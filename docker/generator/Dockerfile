# TODO : find a way to include the main dockerfile instead of copy-pasting it here

FROM        alpine as gitinfo

RUN apk add git
WORKDIR /build

# Write build info to be available at $url/buildinfo
RUN echo "{" > /build/buildinfo
RUN echo "  \"build-date\": \"`date +%s`\"," >> /build/buildinfo
RUN echo "  \"build-date-human\": \"`date`\"," >> /build/buildinfo
RUN echo "}" >> /build/buildinfo

# Final container (copies in /out/buildinfo when done)
FROM        perl:5.28-slim as final
MAINTAINER  Ben Yanke <ben@benyanke.com>

# Set envs
ENV APACHE_RUN_USER www-data \
    APACHE_RUN_GROUP www-data \
    APACHE_LOCK_DIR /var/lock/apache2 \
    APACHE_LOG_DIR /var/log/apache2 \
    APACHE_PID_FILE /var/run/apache2/apache2.pid \
    APACHE_SERVER_NAME localhost

# Install packages
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    apache2 \
    libcgi-session-perl \
    && rm -rf /var/lib/apt/lists/*

# Get dumb-init to use a proper init system
RUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.2/dumb-init_1.2.2_amd64 && \
    chmod +x /usr/local/bin/dumb-init

# Load config files
COPY docker/apache/ports.conf /etc/apache2/ports.conf
COPY docker/apache/apache2.conf /etc/apache2/apache2.conf

# Set permissionsso apache can write to logs without root
RUN mkdir -p /var/run/apache2 /var/lock/apache2 /var/log/apache2 ; chown -R www-data:www-data /var/lock/apache2 /var/log/apache2 /var/run/apache2

# Copy in code
WORKDIR /var/www
COPY --chown=www-data:www-data web /var/www/web

# Write build info to be available at $url/buildinfo
COPY --from=gitinfo /build/buildinfo /var/www/web/buildinfo

# Expose default port
EXPOSE 80

# Install packages
RUN apt-get update && apt-get install -y \
    bsdmainutils \
    zip \
    libwww-perl \
    && rm -rf /var/lib/apt/lists/*


WORKDIR /var/www/standalone/tools/epubgen2/
CMD ["/bin/bash", "/var/www/standalone/tools/epubgen2/epubgen2.sh"]
