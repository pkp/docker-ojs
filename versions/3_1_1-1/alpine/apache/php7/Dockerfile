FROM alpine:3.13.6

LABEL maintainer="Public Knowledge Project <marc.bria@gmail.com>"
WORKDIR /var/www/html

ENV COMPOSER_ALLOW_SUPERUSER=1  \
    SERVERNAME="localhost"      \
    HTTPS="on"                  \
    OJS_VERSION=ojs-3_1_1-1 \
    OJS_CLI_INSTALL="0"         \
    OJS_DB_HOST="localhost"     \
    OJS_DB_USER="ojs"           \
    OJS_DB_PASSWORD="ojs"       \
    OJS_DB_NAME="ojs"           \
    OJS_WEB_CONF="/etc/apache2/conf.d/ojs.conf"   \
    OJS_CONF="/var/www/html/config.inc.php"       \
    PACKAGES="supervisor dcron ttf-freefont apache2 apache2-ssl apache2-utils patch php7 php7-fpm php7-cli php7-apache2   \
             php7-zlib php7-json php7-mbstring php7-tokenizer php7-simplexml php7-phar php7-openssl    \
             php7-curl php7-mcrypt php7-pdo_mysql php7-mysqli php7-session php7-ctype php7-gd php7-xml \
             php7-dom php7-iconv curl nodejs git npm" \
    EXCLUDE_URL="https://raw.githubusercontent.com/pkp/docker-ojs/master/excludeVar.list"

RUN apk add --update --no-cache $PACKAGES && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    # Configure and download code from git
    git config --global url.https://.insteadOf git:// && \
    git config --global advice.detachedHead false && \
    git clone --depth 1 --single-branch --branch $OJS_VERSION --progress https://github.com/pkp/ojs.git . && \
    git submodule update --init --recursive >/dev/null

# Install NPM and Composer Deps
RUN composer update -d lib/pkp --no-dev && \
    composer install -d plugins/paymethod/paypal --no-dev && \
    composer install -d plugins/generic/citationStyleLanguage --no-dev && \
    npm install && npm run build

# Create directories
RUN mkdir -p /var/www/files /run/apache2  /run/supervisord/ && \
    cp config.TEMPLATE.inc.php config.inc.php && \
    chown -R apache:apache /var/www/* && \
    # Prepare freefont for captcha 
	ln -s /usr/share/fonts/TTF/FreeSerif.ttf /usr/share/fonts/FreeSerif.ttf \
    # Prepare crontab
    echo "0 * * * *   ojs-run-scheduled" | crontab - && \
    # Prepare httpd.conf
    sed -i -e '\#<Directory />#,\#</Directory>#d' /etc/apache2/httpd.conf && \
    sed -i -e "s/^ServerSignature.*/ServerSignature Off/" /etc/apache2/httpd.conf

# Clear the image (list of files to be deleted in exclude.list).
COPY exclude.list /tmp/exclude.list
RUN rm -rf $(cat /tmp/exclude.list) && \
    apk del --no-cache nodejs git && \
    find . \( -name .gitignore -o -name .gitmodules -o -name .keepme \) -exec rm '{}' \;


COPY root/ /

EXPOSE 80 443

VOLUME [ "/var/www/files", "/var/www/html/public" ]

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
