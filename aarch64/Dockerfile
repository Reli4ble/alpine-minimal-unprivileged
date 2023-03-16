FROM scratch AS builder

ARG ARCH=aarch64
ARG ALPINE_VERSION=3.17.2
ADD alpine-minirootfs-${ALPINE_VERSION}-${ARCH}.tar.gz /

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]

# ---------------------------------------------------------------------
# Install necessary packages to install nginx
# ---------------------------------------------------------------------
RUN apk add openssl curl ca-certificates --no-cache && \
# ---------------------------------------------------------------------
# Set up the apk repository for the stable-line nginx packages
# ---------------------------------------------------------------------
    printf "%s%s%s\n" "http://nginx.org/packages/alpine/v" `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` "/main" | tee -a /etc/apk/repositories && \
# ---------------------------------------------------------------------
# Import an official nginx signing key so apk could verify the packages authenticity. Fetch the key:
# ---------------------------------------------------------------------   
    curl -o /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub && \
# ---------------------------------------------------------------------
# Verify that the downloaded file contains the proper key:
# ---------------------------------------------------------------------     
    openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout && \
# ---------------------------------------------------------------------
# Move the key to apk trusted keys storage:
# ---------------------------------------------------------------------     
    mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/ && \
# ---------------------------------------------------------------------
# Install nginx and tzdata and rm curl
# ---------------------------------------------------------------------     
    apk add nginx tzdata --no-cache && \
    apk del curl --no-cache && \
# ---------------------------------------------------------------------
# Mod the confs
# --------------------------------------------------------------------- 
    sed -i 's,listen       80;,listen       8080;,' /etc/nginx/conf.d/default.conf && \
    sed -i '/^user  nginx;/d' /etc/nginx/nginx.conf && \
    sed -i 's,/var/run/nginx.pid,/tmp/nginx.pid,' /etc/nginx/nginx.conf && \
    sed -i "/^http {/a \    proxy_temp_path /tmp/proxy_temp;\n    client_body_temp_path /tmp/client_temp;\n    fastcgi_temp_path /tmp/fastcgi_temp;\n    uwsgi_temp_path /tmp/uwsgi_temp;\n    scgi_temp_path /tmp/scgi_temp;\n" /etc/nginx/nginx.conf && \
# ---------------------------------------------------------------------    
# nginx user must own the cache and etc directory to write cache and tweak the nginx config
# ---------------------------------------------------------------------
    chown -R nginx:0 /var/cache/nginx && \
    chmod -R g+w /var/cache/nginx && \
    chown -R nginx:0 /etc/nginx && \
    chmod -R g+w /etc/nginx && \
# ---------------------------------------------------------------------    
# forward request and error logs to stdout
# ---------------------------------------------------------------------
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log   
# ---------------------------------------------------------------------    
# Building the final image
# ---------------------------------------------------------------------
FROM scratch
COPY --from=builder / /
USER nginx
ENV TZ="Europe/Berlin"
EXPOSE 8080
STOPSIGNAL SIGQUIT
CMD ["nginx", "-g", "daemon off;"]
