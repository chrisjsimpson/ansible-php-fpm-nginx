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
ansible-playbook -i inventory.ini playbooks/nginx/nginx.yaml;
ansible-playbook -i inventory.ini playbooks/php-fpm.yaml;
ansible-playbook -i inventory.ini playbooks/mysql/mysql.yaml; # see configuration notes
ansible-playbook -i inventory.ini playbooks/mongo/mongo.yaml
ansible-playbook -i inventory.ini playbooks/beanstalkd/beanstalkd.yaml
ansible-playbook -i inventory.ini playbooks/redis/redis.yaml
ansible-playbook -i inventory.ini playbooks/php-redis/php-redis.yaml

# Or, run all playbooks in one go:
ansible-playbook -i inventory.ini playbooks/main/main.yaml
```

## Configuration

Each playbook has a `vars/` directory for config. 

### Mysql setting database name and password
Rather than hard-code passwords in the repo (see `playbooks/mysql/vars/main.yam` for default
user and database created), you can specify the database(s) and user(s) to be created at
the command line for example:

Ensure mysql has a database created called `test_db` with a user will all permissions
to that database with the username `test_user` and password `password`
```
ansible-playbook -i inventory.ini playbooks/mysql/mysql.yaml --extra-vars='{
  "mysql_databases": [
    {
      "name": "test_db",
      "encoding": "utf8",
      "collation": "utf8_general_ci"
    }
  ],
  "mysql_users": [
    {
      "name": "test_user",
      "host": "%",
      "password": "password",
      "priv": "test_db.*:ALL"
    }
  ]
}'
```

## Manual steps :( 

Summary
*Note* Some of these steps should/coud be handles by a pipeline

- Clone the php repo (e.g. a Laravel repo)
- Run composer install (yes, as root for now)
- Copy .env file over
- Generate or set application get (Laravel only e.g. `php artisan key:generate`)
- Set ownership to www-data (because php-fpm is running as the `www-data` user)

### Clone repo

Clone the php repo in the empty html folder: /var/www/html

1. `git clone <repo> /var/www/html` 
If you can't clone: Generate a key (`ssh-keygen`) and add `cat  ~/.ssh/id_rsa.pub` to your repo as an allowed key

### Run composer install 

1. `cd /var/www/html`
2. `composer install`

### Env file 

Place the .env file in the `/var/www/html` directory 

### Set ownership of the repo to www-data

1. cd /var/www/html
2. chown -R chown -R www-data ./
3. Restart php-fpm `systemctl restart php-fpm.service`
