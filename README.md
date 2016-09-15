# ansible-guacamole-server

Current guacamole version: **0.9.9**

Current tomcat version: **tomcat7** (Ubuntu)

---
### Role variables
| Variable                | Required | Default | Choices                   | Comments                                 |
|-------------------------|----------|---------|---------------------------|------------------------------------------|
| GUACD_PORT              | yes      | 4822    | any port                  | use in `guacamole.properties` template   |
| GUACAMOLE_HOME          | yes      | /usr/share/tomcat7/.guacamole |                | [Read this if you want to move `GUACAMOLE_HOME`](http://guacamole.incubator.apache.org/doc/gug/configuring-guacamole.html)|
| server_addr             | yes      | localhost:8080/guacamole  | hostname of the server | This is used to get the beginning of the URL by the URL generator|
| addrs                   | yes      | localhost| any IPs you want to connect to| used in `gen_guac_url.py` template|
| protocols               | yes      | vnc,ssh | vnc, ssh                  | used in `gen_guac_url.py` template to specify the connection types we want for each addr|
| hmac_secret             | yes      | secret  | any string                | HMAC auth relies on this, so make it good!                         |
| hmac_age_limit          | yes      | 600,000 seconds | time in seconds   | used in `gucamole.properties` template to set how long a generated URL is valid|
---
### Authentication
[Guacamole authentication is performed with HMAC](https://github.com/calvinmclean/guacamole-auth-hmac). This allows connections to be accessed without user accounts. The code for HMAC includes a python script to create the links that will look like this:

`http://localhost:8080/guacamole/#/c/client/4415?timestamp=1473789652637&guac.port=5901&guac.password=PASSWORD&guac.protocol=vnc&signature=f5bmkMtP4x6T0yODsptHLyOf8xI%2Bth%2FNs2sTa83QCWY%3D&guac.hostname=HOSTNAME&id=c%2F4415`

##### URL creation
The URL generator concatenates the secret, the timestamp, and the protocol and hashes this string with sha256. This is then encoded with base64 to produce the 'signature' part of the URL.

The links are created with a timestamp of their creation times, and the server has an age-limit property so the links expire. This requires generation of new links, or setting age-limit to unlimited.

This authentication method is useful in this application of Guacamole because it does not require additional user accounts, and the link takes the user directly to the connection they need.

#### SSH
SSH is authenticated using the users' Unix usernames and passwords as they would normally.

SSH can be authenticated using a keypair. The server would have its RSA private key and each VM would have the public key added during setup.

#### VNC
VNC is authenticated using the VNC password that is set during VNC server setup on the client VM.

The problem with this is that, as you can see in the link above, the password is visible. Although knowing the password is useless as long as the user cannot create a valid signature for other connections, it is still not good to have it show for all users. My proposed fix for this is to either:

1. Use a random password for each new VM. This password will have to be remembered by the server so it can be reused on creation of new links.
  - Implementation: when setting up the VMs, add a task for the Ansible role to send this password to the server's database, then the server will get the password by querying the client's ID when creating the link.
  - Problem: managing the database will be difficult because the passwords should be removed when the client instance is deleted. Also, it is difficult to query by connection IDs, especially if DB cleaning is flawed and the DB gets cluttered.
2. Use a password for each user. These passwords will be remembered by the server and used for creating the new links.
  - Implementation: when setting up VMs, add a task to the Ansible role to query the server's database to receive the user's VNC password.
  - Problem: user passwords should be changed out after some time, but this is easy to implement.

Guacamole does not currently support encrypted connections from RealVNC because it is a non-standard VNC feature. Guacamole developers want to add this feature, but there is no expected release, and it will probably be a while.

---
### Example Playbook
```
- hosts: guac-server
  remote_user: root
  roles:
    - ansible-guacamole-server
```
---
### Issue & Roadmap
- Depending on which method of VNC login will be used, I will have to add some stuff. For both, I will need to set up a database in this role.
  - Option 1: add a task to get connection password from the database and add new password for new connections
  - Option 2: add a task to get user VNC passwords from the database
- Where does the generation of new links occur? How will they be updated on the Atmosphere UI?
  - I think that when a new VM is launched, it should have the `gen_guac_url.py` itself, since generating a link does not require anything from the server. However, this leaves the HMAC secret vulnerable.
- I will likely need a separate role for adding connections to the server, since it doesn't need to be configured each time, or use `--tags=vars,template` with this role.
