FROM php:7.1-fpm

MAINTAINER pierre@piwik.org

RUN set -ex; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libgeoip-dev \
		libjpeg-dev \
		libldap2-dev \
		libpng12-dev \
	; \
	\
	debMultiarch="$(dpkg-architecture --query DEB_BUILD_MULTIARCH)"; \
	docker-php-ext-configure gd --with-freetype-dir=/usr --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-configure ldap --with-libdir="lib/$debMultiarch"; \
	docker-php-ext-install \
		gd \
		ldap \
		mysqli \
		opcache \
		pdo_mysql \
		zip \
	; \
	\
	pecl install \
		APCu-5.1.9 \
		geoip-1.1.1 \
		redis-3.1.6 \
	; \
	\
	docker-php-ext-enable \
		apcu \
		geoip \
		redis \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

ENV PIWIK_VERSION 3.3.0

RUN set -ex; \
	curl -fsSL -o piwik.tar.gz \
		"https://builds.piwik.org/piwik-${PIWIK_VERSION}.tar.gz"; \
	curl -fsSL -o piwik.tar.gz.asc \
		"https://builds.piwik.org/piwik-${PIWIK_VERSION}.tar.gz.asc"; \
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --keyserver ha.pool.sks-keyservers.net --recv-keys 814E346FA01A20DBB04B6807B5DBD5925590A237; \
	gpg --batch --verify piwik.tar.gz.asc piwik.tar.gz; \
	rm -r "$GNUPGHOME" piwik.tar.gz.asc; \
	tar -xzf piwik.tar.gz -C /usr/src/; \
	rm piwik.tar.gz

COPY php.ini /usr/local/etc/php/php.ini

RUN set -ex; \
	curl -fsSL -o /usr/src/piwik/misc/GeoIPCity.dat.gz https://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz; \
	gunzip /usr/src/piwik/misc/GeoIPCity.dat.gz

COPY docker-entrypoint.sh /entrypoint.sh

# WORKDIR is /var/www/html (inherited via "FROM php")
# "/entrypoint.sh" will populate it at container startup from /usr/src/piwik
VOLUME /var/www/html

ENTRYPOINT ["/entrypoint.sh"]
CMD ["php-fpm"]
