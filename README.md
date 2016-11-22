# ansible-guacamole-server

*Master:* [![Build Status](https://travis-ci.org/calvinmclean/ansible-guacamole-server.svg?branch=master)](https://travis-ci.org/calvinmclean/ansible-guacamole-server) *Dev:* [![Build Status](https://travis-ci.org/calvinmclean/ansible-guacamole-server.svg?branch=dev)](https://travis-ci.org/calvinmclean/ansible-guacamole-server)

Current guacamole version: **0.9.9**

Current tomcat version: **7**

---
### Role variables
| Variable                | Required | Default | Choices                   | Comments                                 |
|-------------------------|----------|---------|---------------------------|------------------------------------------|
| GUACD_PORT              | yes      | 4822    | any port                  | use in `guacamole.properties` template   |
| GUACAMOLE_HOME          | yes      | /usr/share/tomcat7/.guacamole |                | [Read this if you want to move `GUACAMOLE_HOME`](http://guacamole.incubator.apache.org/doc/gug/configuring-guacamole.html)|
| server_addr             | yes      | localhost  | hostname of the server | used in `guacamole.properties`|
| hmac_secret             | yes      | secret  | any string                | HMAC auth relies on this, so make it good!                         |
| hmac_age_limit          | yes      | 600,000 seconds | time in seconds   | used in `gucamole.properties` template to set how long a generated URL is valid|
---
### Authentication
[Guacamole authentication is performed with HMAC](https://github.com/calvinmclean/guacamole-auth-hmac). This allows connections to be accessed without user accounts. The code for HMAC includes form.php that will send the authentication information as a POST request and redirect the user to their connection with the returned authToken. The URL looks like:

`http://localhost:8888/guacamole/#/client/ZjA4YzRlLWM3MWUtNDE1OC05MTNkLTlkYjRlNzEyYmJkNwBjAGhtYWM=?token=bd786cd40e307d5c5b14bde0dc5fffcd9f797cfb64ae0c1947c247054d80efac`

Read more about this in **[HMAC-README.md](https://github.com/calvinmclean/guacamole-auth-hmac)**

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
