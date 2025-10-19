FROM kahidna/docker-phpfpm:8.4

MAINTAINER Alfin Hidayat <kahid.na@gmail.com>

ENV OS_LOCALE="en_US.UTF-8" \
    DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y locales && locale-gen ${OS_LOCALE}
ENV LANG=${OS_LOCALE} \
    LANGUAGE=${OS_LOCALE} \
    LC_ALL=${OS_LOCALE} \
	NGINX_CONF_DIR=/etc/nginx

COPY ./supervisord.conf /etc/supervisor/conf.d/
COPY ./app /var/www/app/

RUN	\
	BUILD_DEPS='wget gnupg' \
    && dpkg-reconfigure locales \
	&& apt-get install --no-install-recommends -y $BUILD_DEPS \
        && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
        && locale-gen \
    && wget -O - http://nginx.org/keys/nginx_signing.key > /tmp/nginx.key &&  gpg --no-default-keyring --keyring /tmp/nginx_keyring.gpg --import /tmp/nginx.key  \
&& gpg --no-default-keyring --keyring /tmp/nginx_keyring.gpg --export > /tmp/nginx.gpg && mv /tmp/nginx.gpg /etc/apt/trusted.gpg.d/ \
	&& echo "deb http://nginx.org/packages/debian/ trixie nginx" | tee -a /etc/apt/sources.list \
	&& echo "deb-src http://nginx.org/packages/debian/ trixie nginx" | tee -a /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install -y nginx \
	&& rm -rf ${NGINX_CONF_DIR}/sites-enabled/* ${NGINX_CONF_DIR}/sites-available/* \
	# Install supervisor
	&& apt-get install -y supervisor && mkdir -p /var/log/supervisor \
	&& chown www-data:www-data /var/www/app/ -Rf \
	# Cleaning
	&& apt-get purge -y --auto-remove $BUILD_DEPS \
	&& apt-get autoremove -y && apt-get clean \
	&& rm -rf /var/lib/apt/lists/* \
	# Forward request and error logs to docker log collector
	&& ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

COPY ./configs/nginx.conf ${NGINX_CONF_DIR}/nginx.conf
COPY ./configs/app.conf ${NGINX_CONF_DIR}/sites-enabled/app.conf
COPY ./configs/www.conf /etc/php/8.4/fpm/pool.d/www.conf

WORKDIR /var/www/app/

EXPOSE 80 443

CMD ["/usr/bin/supervisord"]
