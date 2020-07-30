# ansible-guacamole-server
This role will install and configure Guacamole for CyVerse using Docker Compose.


### Role variables
| Variable              | Default                        | Description |
|-----------------------|--------------------------------|-------------|
| `CONFIG_DIR`          | `"/opt/guacamole"`             | this is the root of everything that will be configured on the target |
| `NGINX_DIR`           | `"{{ CONFIG_DIR }}/nginx"`     | this is where we will setup Nginx configs and keys |
| `GUACAMOLE_DIR`       | `"{{ CONFIG_DIR }}/guacamole"` | this is where we will setup all Guacamole configs |
| `install_type`        | `"cyverse"`                    | this determines which theme extension to use. Can be: `"cyverse"`, `"jetstream"` |
| `guacamole_version`   | `"1.0.0"`                      | used to tag Guacamole images and download authentication plugin |
| `guacd_ssl`           | `false`                        | used in `guacamole.properties` template. Also enables extra tasks to configure SSL |
| `secret_key`          | `"secret"`                     | HMAC auth relies on this, so make it good! |
| `timestamp_age_limit` | `600000`                       | used in `gucamole.properties` template to set how long a generated URL is valid (in milliseconds) |
| `key_directory`       | `"{{ GUACAMOLE_DIR }}/keys"`   | this is where we store user keypairs for SSH connections |
| `use_local_privkey`   | `true`                         | determine if the authentication plugin should look for SSH keys on local filesystem |
| `GUACD_LOG_LEVEL`     | `"info"`                       | logging level for `guacd` container. Can be one of: `"trace"`, `"debug"`, `"info"`, `"warning"`, `"error"` |
| `ssh_port`            | `22`                           | Port used to SSH to the server so that UFW allows it |
| `nginx_port`          | `80`                           | Port used by Nginx for HTTP |
| `nginx_https_port`    | `443`                          | Port used by Nginx for HTTPS |
| `server_addr`         | `"localhost"`                  | used in Nginx configuration |
| `proxy_path`          | `"/"`                          | path to use in URL for proxying to Guacamole container |
| `ssl_cert_name`       | `"fullchain.pem"`              | name of the SSL certificate to be used for copying files and Nginx config |
| `ssl_key_name`        | `"privkey.pem"`                | name of the SSL keyfile to be used for copying files and Nginx config |
| `local_ssl_path`      | undefined                      | local path to point to user-provided SSL files. If it is a relative path, start with "." |


## Usage
1. Download this repository
    ```bash
    git clone https://github.com/CyVerse-Ansible/ansible-guacamole-server.git
    ```

2. Install requirements
    ```bash
    ansible-galaxy role install -r ansible-guacamole-server/requirements.yml
    ```

3. Create `playbook.yml`
    ```yaml
    - hosts: guac-server
      vars_files:
        - config.yml
      roles:
        - docker
        - ansible-guacamole-server
    ```

4. Create `hosts.yml`
    ```yaml
    all:
      hosts:
        guac-server:
          ansible_host: localhost
          ansible_user: root
    ```

5. Create `config.yml` (these are commonly-overridden variables)
    ```yaml
    server_addr: guacamole.cyverse.org
    local_ssl_path: "./files"
    ssl_cert_name: "cyverse.org.pem"
    ssl_key_name: "cyverse.key"
    ```

6. Run it!
    ```bash
    ansible-playbook -i hosts.yml playbook.yml
    ```
