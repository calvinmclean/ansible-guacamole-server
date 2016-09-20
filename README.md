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

The python file in templates is used to create URLs and can be used like this: `python gen_guac_url.py [HOSTNAME] [VNCPASS] [PROTOCOL]`

#### SSH
SSH is authenticated using the users' Unix usernames and passwords as they would normally.

SSH can be authenticated using a keypair. The server would have its RSA private key and each VM would have the public key added during setup.

#### VNC
VNC is authenticated using the VNC password that is set during VNC server setup on the client VM.

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
- Where does the generation of new links occur? How will they be updated on the Atmosphere UI?
  - I think that when a new VM is launched, it should have the `gen_guac_url.py` itself, since generating a link does not require anything from the server. However, this leaves the HMAC secret vulnerable.
- I will likely need a separate role for adding connections to the server, since it doesn't need to be configured each time, or use `--tags=vars,template` with this role.
