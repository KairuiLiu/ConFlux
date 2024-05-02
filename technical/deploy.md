# Deployment Guide

## Installing Packages (CentOS example)

- Install common dependencies (steps omitted)
  - Nginx
  - Node.js (>=18)
  - pnpm
  - HTTPS certificate

- Install and configure [coturn](https://github.com/coturn/coturn)

  ```bash
  # Install coturn
  yum install coturn

  # Create configuration file
  vim /etc/turnserver.conf
  # Write the following
  # listening-ip=0.0.0.0

  # Start coturn and enable it to start on boot
  systemctl enable --now coturn
  ```

- Install and configure [MongoDB](https://www.mongodb.com/)

  ```bash
  # Set up repository according to https://www.mongodb.com/docs/manual/administration/install-on-linux/

  # Install MongoDB
  yum install mongodb-org

  # Edit the configuration file
  vim /etc/mongod.conf
  # Modify
  # security:
  #   authorization: enabled

  # Start MongoDB and enable it to start on boot
  systemctl enable --now mongod

  # Create users
  mongosh
  > use admin
  > db.createUser({ user: "admin", pwd: "your_password", roles: [{ role: "root", db: "admin" }] });
  > db.createUser({ user: "conflux", pwd: "conflux", roles: [{ role: "readWrite", db: "conflux" }] });
  > use conflux
  > exit

  # Attempt to log in
  > mongosh "mongodb://conflux:conflux@localhost:27017/conflux?authSource=admin"
  ```

## Deployment

```bash
mkdir /path/to/wwwroot/conflux
```

- Download and build the frontend repository

  ```bash
  cd /path/to/wwwroot/conflux
  # Download frontend code
  git clone --depth 1 -b master git@github.com:KairuiLiu/conflux-client.git
  cd conflux-client

  # Install dependencies
  pnpm i

  # Build
  pnpm build
  mv dist ../html
  ```

- Download and run backend code

  ```bash
  cd /path/to/wwwroot/conflux
  # Download backend code
  git clone --depth 1 -b master git@github.com:KairuiLiu/conflux-server.git
  cd conflux-server

  # [Optional] Edit `.env` file

  # Install dependencies
  pnpm i
  pnpm i -g pm2

  # Build
  pnpm build
  mv dist ../server

  # Start
  pm2 start /path/to/wwwroot/conflux/server/ecosystem.config.cjs
  ```

## Nginx Configuration

- Configure Nginx

  ```nginx
  http {
    # ...
    types {
        application/wasm wasm;
    }
  }
  ```

  ```nginx
  server {
    listen 443 ssl http2;

    server_name  YOUR_DOMAIN_NAME;
    index index.html index.htm;

    root  /path/to/wwwroot/conflux/html;
    try_files $uri $uri/ /index.html

    error_page 404 /index.html;
    ssl_certificate /path/to/wwwroot/ssl/cert.pem;
    ssl_certificate_key /path/to/wwwroot/ssl/key.pem;

    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_min_length 256;
    gzip_types application/atom+xml application/geo+json application/javascript application/x-javascript application/json application/ld+json application/manifest+json application/rdf+xml application/rss+xml application/xhtml+xml application/xml font/eot font/otf font/ttf image/svg+xml text/css text/javascript text/plain text/xml application/wasm;

    location /socket.io/ {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $host;

      proxy_pass http://localhost:9876;
      proxy_read_timeout 86400;

      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }

    location /api/peer_signal {
      rewrite ^/api(/peer_signal.*)$ $1 break;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $host;

      proxy_pass http://localhost:9877;
      proxy_read_timeout 86400;

      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }

    location /api/ {
      proxy_pass http://127.0.0.1:9876/;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_read_timeout 86400;
    }

    location ^~ /tflite/ {
      expires 365d;
    }

    location ~* (jpe?g|png|gif|svg) {
      expires      30d;
      log_not_found off;
      error_page 404 /content/images/system/default/404.gif;
    }

    location ~ \.(ttf|ttc|otf|eot|woff|woff2|font.css)$ {
      add_header Access-Control-Allow-Origin "*";
      expires      30d;
    }

    location /nginx_status {
      stub_status on;
      access_log   off;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
      expires      12h;
    }

    location ~ .*\.(js|css)?$ {
      expires      12h;
    }

    location ~ /.well-known {
      allow all;
    }

    location ~ /\. {
      deny all;
    }

    access_log  /var/log/nginx/access_conflux.log;
    error_log /var/log/nginx/error_conflux.log;
  }
  ```

- Attempt to start Nginx

  ```bash
  nginx -t
  systemctl reload nginx
  ```
