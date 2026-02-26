ARG WD="/usr/share/nginx/html"

FROM alpine:3.23 AS base

ARG WD

ENV SHELL=/bin/sh

RUN apk --no-progress --purge --no-cache upgrade \
 && apk --no-progress --purge --no-cache add --upgrade \
    curl \
    nginx \
    openssl \
    sudo \
    tzdata \
 && rm -vrf /var/cache/apk/*

# Set Timezone
ENV TZ=America/Detroit
# Sync clock with NTP
RUN cp /usr/share/zoneinfo/America/Detroit /etc/localtime

# Comment out a line that sets the user.
RUN sed -i 's/user nginx;/#&/' /etc/nginx/nginx.conf

# Send the logs directly to STDOUT and STDERR
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
 && ln -sf /dev/stderr /var/log/nginx/error.log

# Allow the nginx user to use sudo without entering a password.
RUN echo 'nginx ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/nginx

# Download a script to generate an SSL certificate.
RUN curl -L -o gen-ss-cert.sh https://raw.githubusercontent.com/b01/script-lib/refs/tags/0.1.1/generate-self-signed-cert.sh \
 && chmod +x ./gen-ss-cert.sh \
 && mv gen-ss-cert.sh /usr/local/bin

WORKDIR "${WD}"

USER nginx

RUN sudo gen-ss-cert.sh --company="containers" \
    --sans="DNS:localhost,DNS:ip6-loopback,IP:127.0.0.1,IP:0:0:0:0:0:0:0:1" \
    --out-dir="${HOME}/pki" \
    --cert-ext="crt" \
    "local.dev"

# Make nginx server run as the main container process.
ENTRYPOINT [ "nginx" ]

# example nginx useage: /usr/bin/nginx -t -c ~/mynginx.conf -g "pid /var/run/nginx.pid; worker_processes  `sysctl -n hw.ncpu`;"

# Configure nginx to run in the foreground, which keeps the container running.
CMD [ "-g", "daemon off;" ]

EXPOSE 443/tcp

HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f https://localhost/ || exit 1

COPY --chmod=644 .config/docker/frontend/default.conf /etc/nginx/http.d/default.conf
