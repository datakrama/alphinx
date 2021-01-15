FROM docker.io/library/alpine:3.13
LABEL Maintainer="Tim de Pater <code@trafex.nl>" \
      Description="Lightweight container with Nginx 1.18 & PHP-FPM 7.3 based on Alpine Linux."

# Install packages and remove default server definition
RUN apk --no-cache add php8 php8-fpm php8-opcache \
    nginx supervisor curl && \
    rm /etc/nginx/conf.d/default.conf

# Configure nginx
COPY ../../config/nginx.conf /etc/nginx/nginx.conf

# Configure PHP-FPM
COPY ../../config/fpm-pool.conf /etc/php8/php-fpm.d/www.conf
COPY ../../config/php.ini /etc/php8/conf.d/custom.ini

# Configure supervisord
COPY ../../config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/html

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN apk --no-cache add shadow && \
    usermod -u 1000 nobody && \
    groupmod -g 1000 nobody

RUN chown -R nobody.nobody /var/www/html && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html
COPY --chown=nobody ../../src/ /var/www/html/

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping