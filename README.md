# NGINX unprivileged image based on alpine-minirootfs

This repo contains the dockerfiles to build NGINX from the Stable-Version on the latest alpine-minirootfs.

All Dockerfiles uses only Multistage-Builds with Scratch-Images and including only that is needed to run NGINX.

NGINX runs here really unprivileged that means master and worker-processes runs under unprivileged user-accounts.


The alpine-minirootfs is getting from here: https://github.com/alpinelinux/docker-alpine.

NGINX is getting only from the Stable-Branch.


# Supported Image Platforms and Registries

### GitHub Container Registry:

- x86_64: https://github.com/Reli4ble/nginx-minimal-unprivileged/pkgs/container/nginx-minimal-unprivileged-x86_64
- aarch64: https://github.com/Reli4ble/nginx-minimal-unprivileged/pkgs/container/nginx-minimal-unprivileged-aarch64

### Platforms:

This NGINX image is only provided for the following platforms

`x86_64`  `aarch64`

# Usage:


# Config:

This NGINX image is delivered with the standard config for a web server but has also received small changes to be able to run completely unprivileged in a container:

- removed `user  nginx;` from /etc/nginx/nginx.conf but this makes no sense
- move `/var/run/nginx.pid` to `/tmp/nginx.pid`
- all nginx-processes - master & worker runs under user nginx
- access & error-logs linked to `/dev/stdout`
- default http-port changed to `8080`
- changed owner/mod of nginx etc & cache-directory to run unprivileged

### /etc/nginx/nginx.conf:
```
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /tmp/nginx.pid;


events {
    worker_connections  1024;
}


http {
    proxy_temp_path /tmp/proxy_temp;
    client_body_temp_path /tmp/client_temp;
    fastcgi_temp_path /tmp/fastcgi_temp;
    uwsgi_temp_path /tmp/uwsgi_temp;
    scgi_temp_path /tmp/scgi_temp;

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

### /etc/nginx/conf.d/default.conf:
```
server {
    listen       8080;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
