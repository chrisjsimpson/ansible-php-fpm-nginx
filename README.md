# PHP-FPM + Nginx Ansible

Configure a server with php-fpm and nginx. This saves time from having to set-up a server manually.

It leaves the server in a state where the `/var/www/html` directory is empty
ready to be populated by a php project, e.g. via a pipeline.

Details:

- Installs php from source, so can easily keep php up to date
  - See var `php_source_version` to build & install another version of php (see playbooks/php/vars/main.yaml)
  - You can override this at install time, using `-e php_source_version=php-7.4.8` flag on `ansible-playbook`
- Nginx is configured to point to fpm running on 127.0.0.1:900
- The site directory is /var/www/html

# Setup

Create an instance (e.g. linode vps) copy its ip address
**Note**: This assumes an Ubuntu based image, though any debian based image may work too.

Update inventory.yaml with the ip address of the target server (add multiple ips, one on each line, if you
need to set-up multiple servers).

Install ansible requirements (roles)

```
ansible-galaxy install -r requirements.yml
```

## Verify access using ansible ping

For example, if your vps ip address is `178.79.190.75`

```
ansible -i inventory.ini targets -m ping
The authenticity of host '178.79.190.75 (178.79.190.75)' can't be established.
ECDSA key fingerprint is SHA256:85UXi/Dpo+6v/9ZMVuMhBKSbVrxeUwp0azKkMw/kDHk.
Are you sure you want to continue connecting (yes/no)? yes
178.79.190.75 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
  
# Installation of php with fpm & nginx

- Install ansible
- run `ansible-galaxy install -r requirements.yml`

# Run
Run the playbooks

- Installs php with php-fpm 
- Installs nginx with default site using php-fpm's engine

```
ansible-playbook -i inventory.ini playbooks/php-fpm.yaml; 
ansible-playbook -i inventory.ini playbooks/nginx/nginx.yaml;
```
