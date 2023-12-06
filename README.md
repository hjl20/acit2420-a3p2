# ACIT2420 A3P2 - README

# File directories

hello-server and index.html

```bash
/var/www/my-site
```

hello-server.service

```bash
/etc/systemd/system
```

contents:

```bash
[Unit]
Description=A web api that says hello
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=www-data
Restart=on-failure
ExecStart=/var/www/my-site/hello-server

StandardOutput=append:/var/log/hello-server/backend.log
StandardError=append:/var/log/hello-server/backend.log
SyslogIdentifier=hello-server

[Install]
WantedBy=multi-user.target
```

**You must create `/var/log/hello-server/backend.log` manually!**

hello.conf

```bash
/etc/nginx/sites-available
```

with the following symlink:

```bash
sudo ln -s /etc/nginx/sites-available/hello.conf /etc/nginx/sites-enabled/hello.conf
```

contents:

```bash
server {
    listen 80;
    listen [::]:80;

    root /var/www/my-site;

    index index.html;

    server_name _;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    location /hey {
        # Define the reverse proxy settings
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /echo {
        # Define the reverse proxy settings
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

******************************************You must delete the default nginx site and restart the service.******************************************

# cloud-config.yaml script for initializing droplet

Creates a ‘web’ user and does the following: 

- adds them to the ‘sudo’ group,
- defaults with bash as its login shell,
- logs journal entries for services automatically, and
- removes ‘root’ user SSH access.

```yaml
#cloud-config
users:
  - name: web
    primary_group: web
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - [removed]

packages:
  - ripgrep
  - rsync

runcmd:
  - sed -i 's/^#\(Storage=auto\)/\1/' /etc/systemd/journald.conf
  - sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin no/' /etc/ssh/sshd_config
  - systemctl restart ssh
  - systemctl restart systemd-journald
```
