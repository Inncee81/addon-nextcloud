ARG BUILD_FROM=ghcr.io/hassio-addons/base/aarch64:10.0.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Copy root filesystem
COPY rootfs /

# install packages
RUN \
  echo "**** install build packages ****" && \
  apk add --no-cache \
    apache2-utils \
    git \
    libressl3.3-libssl \
    logrotate \
    nano \
    nginx \
    openssl \
    php7 \
    php7-fileinfo \
    php7-fpm \
    php7-json \
    php7-mbstring \
    php7-openssl \
    php7-session \
    php7-simplexml \
    php7-xml \
    php7-xmlwriter \
    php7-zlib && \
  echo "**** configure nginx ****" && \
  echo 'fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;' >> \
    /etc/nginx/fastcgi_params && \
  rm -f /etc/nginx/http.d/default.conf && \
  echo "**** fix logrotate ****" && \
  sed -i "s#/var/log/messages {}.*# #g" /etc/logrotate.conf && \
  sed -i 's#/usr/sbin/logrotate /etc/logrotate.conf#/usr/sbin/logrotate /etc/logrotate.conf -s /config/log/logrotate.status#g' \
    /etc/periodic/daily/logrotate

# environment settings
ENV NEXTCLOUD_PATH="/data/www/nextcloud" \
  LD_PRELOAD="/usr/lib/preloadable_libiconv.so" \
  NEXTCLOUD_RELEASE=22

RUN \
 echo "**** install build packages ****" && \
 apk add --no-cache --virtual=build-dependencies --upgrade \
	autoconf \
	automake \
	file \
	g++ \
	gcc \
	make \
	php7-dev \
	re2c \
	samba-dev \
	zlib-dev && \
 echo "**** install runtime packages ****" && \
 apk add --no-cache --upgrade \
	curl \
	ffmpeg \
	gnu-libiconv \
	imagemagick \
	libxml2 \
	php7-apcu \
	php7-bcmath \
	php7-bz2 \
	php7-ctype \
	php7-curl \
	php7-dom \
	php7-exif \
	php7-fileinfo \
	php7-ftp \
	php7-gd \
	php7-gmp \
	php7-iconv \
	php7-imagick \
	php7-imap \
	php7-intl \
	php7-ldap \
	php7-mcrypt \
	php7-memcached \
	php7-opcache \
	php7-pcntl \
	php7-pdo_mysql \
	php7-pdo_pgsql \
	php7-pdo_sqlite \
	php7-pgsql \
	php7-phar \
	php7-posix \
	php7-redis \
	php7-sodium \
	php7-sqlite3 \
	php7-xmlreader \
	php7-zip \
	samba-client \
	sudo \
	tar \
	unzip && \
 echo "**** compile smbclient ****" && \
 git clone git://github.com/eduardok/libsmbclient-php.git /tmp/smbclient && \
 cd /tmp/smbclient && \
 phpize7 && \
 ./configure \
	--with-php-config=/usr/bin/php-config7 && \
 make && \
 make install && \
 echo "**** configure php and nginx for nextcloud ****" && \
 echo "extension="smbclient.so"" > /etc/php7/conf.d/00_smbclient.ini && \
 echo 'apc.enable_cli=1' >> /etc/php7/conf.d/apcu.ini && \
 sed -i \
	-e 's/;opcache.enable.*=.*/opcache.enable=1/g' \
	-e 's/;opcache.interned_strings_buffer.*=.*/opcache.interned_strings_buffer=8/g' \
	-e 's/;opcache.max_accelerated_files.*=.*/opcache.max_accelerated_files=10000/g' \
	-e 's/;opcache.memory_consumption.*=.*/opcache.memory_consumption=128/g' \
	-e 's/;opcache.save_comments.*=.*/opcache.save_comments=1/g' \
	-e 's/;opcache.revalidate_freq.*=.*/opcache.revalidate_freq=1/g' \
	-e 's/;always_populate_raw_post_data.*=.*/always_populate_raw_post_data=-1/g' \
	-e 's/memory_limit.*=.*128M/memory_limit=512M/g' \
	-e 's/max_execution_time.*=.*30/max_execution_time=120/g' \
	-e 's/upload_max_filesize.*=.*2M/upload_max_filesize=1024M/g' \
	-e 's/post_max_size.*=.*8M/post_max_size=1024M/g' \
		/etc/php7/php.ini && \
 sed -i \
	'/opcache.enable=1/a opcache.enable_cli=1' \
		/etc/php7/php.ini && \
 echo "env[PATH] = /usr/local/bin:/usr/bin:/bin" >> /etc/php7/php-fpm.conf && \
 echo "**** set version tag ****" && \
 if [ -z ${NEXTCLOUD_RELEASE+x} ]; then \
	NEXTCLOUD_RELEASE=$(curl -s https://raw.githubusercontent.com/nextcloud/nextcloud.com/master/strings.php \
	| awk -F\' '/VERSIONS_SERVER_FULL_STABLE/ {print $2;exit}'); \
 fi && \
 echo "**** download nextcloud ****" && \
 mkdir /app && \
 curl -o /app/nextcloud.tar.bz2 -L \
	https://download.nextcloud.com/server/releases/nextcloud-${NEXTCLOUD_RELEASE}.tar.bz2 && \
 echo "**** cleanup ****" && \
 apk del --purge \
	build-dependencies && \
 rm -rf \
	/tmp/*

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Elcajon" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-on: Elcajon" \
    org.opencontainers.image.authors="Elcajon" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/elcajon/repository-stable" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
