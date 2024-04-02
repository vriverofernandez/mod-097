FROM debian:buster-slim

# Installs and configures Nginx and PHP-fpm

LABEL maintainer="victor rivero"

ENV NGINX_VERSION=1.17.8-1~buster \
    NGINX_VERSION_SHORT=1.17.8 \
    PHP_VERSION=7.4 \
    MODSECCONVER=v1.0.1 \
    MODSECLIBVER=v3.0.4 \
    GEOIP2VER=3.3 \
    HEADERSMOREVER=v0.33 \
    NGXDEVVER=v0.3.1 \
    STREAMLUAVER=master \
    LUANGINXVER=v0.10.15 \
    LUAJIT2VER=v2.1-20200102 \
    RESTYREDISVER=v0.27 \
    RESTYLRUVER=v0.09 \
    RESTYCOREVER=v0.1.17

RUN set -x \
  && apt-get update && apt-get upgrade -y \
  && apt-get install --no-install-recommends --no-install-suggests -y \
    apt-transport-https \
    ca-certificates \
    curl \
    git \
    gnupg1 \
    gzip \
    net-tools \
    sassc \
    tar \
    telnet \
    unzip \
    vim \
    wget \
    rubygems \
    build-essential \
    ruby-dev \
  && \
  set NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
  found='yes'; \
  for server in \
      ha.pool.sks-keyservers.net \
      hkp://keyserver.ubuntu.com:80 \
      hkp://p80.pool.sks-keyservers.net:80 \
      pgp.mit.edu \
  ; do \
      echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
      apt-key adv --keyserver "$server" --keyserver-options timeout=30 --recv-keys found=yes; \
  done; \
  apt-get remove --purge --auto-remove -y \
    gnupg1 \
  && dpkgArch="$(dpkg --print-architecture)" \
  && nginxPackages=" \
     nginx=${NGINX_VERSION} \
  " \
  # we install nginx
  && echo "deb https://nginx.org/packages/mainline/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list \
  && echo "deb https://nginx.org/packages/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list \
  && apt-get update \
  \
  && apt-get install --no-install-recommends --no-install-suggests -y \
      $nginxPackages \
      gettext-base \
  # Finally we clean everything
  && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
  && apt-get autoremove -y && apt-get clean

RUN gem install sass

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stdout /var/log/nginx/error.log


# Compile Nginx modules
RUN \
  # Now is time to compile the modules
  mkdir -p /etc/nginx/modules \
  && mkdir -p /etc/nginx/data \
  && tempDir="$(mktemp -d)" && cd "${tempDir}" \
  && apt-get update \
  && apt-get install --no-install-recommends --no-install-suggests -y \
    dpkg-dev \
  && echo "deb-src https://nginx.org/packages/mainline/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list \
  && echo "deb-src https://nginx.org/packages/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list \
  && apt-get update \
  && apt-get source nginx=${NGINX_VERSION_SHORT} \
  && apt-get install --no-install-recommends --no-install-suggests -y \
    build-essential \
    libpcre3-dev \
    zlib1g-dev \
    libssl-dev \
  && buildArgs="--prefix=/etc/nginx \
      --sbin-path=/usr/sbin/nginx \
      --modules-path=/usr/lib/nginx/modules \
      --conf-path=/etc/nginx/nginx.conf \
      --error-log-path=/var/log/nginx/error.log \
      --http-log-path=/var/log/nginx/access.log \
      --pid-path=/var/run/nginx.pid \
      --lock-path=/var/run/nginx.lock \
      --http-client-body-temp-path=/var/cache/nginx/client_temp \
      --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
      --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
      --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
      --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
      --user=nginx \
      --group=nginx \
      --with-compat \
      --with-file-aio \
      --with-threads \
      --with-http_addition_module \
      --with-http_auth_request_module \
      --with-http_dav_module \
      --with-http_flv_module \
      --with-http_gunzip_module \
      --with-http_gzip_static_module \
      --with-http_mp4_module \
      --with-http_random_index_module \
      --with-http_realip_module \
      --with-http_secure_link_module \
      --with-http_slice_module \
      --with-http_ssl_module \
      --with-http_stub_status_module \
      --with-http_sub_module \
      --with-http_v2_module \
      --with-mail \
      --with-mail_ssl_module \
      --with-stream \
      --with-stream_realip_module \
      --with-stream_ssl_module \
      --with-stream_ssl_preread_module" \
  && ccopt="-g -fdebug-prefix-map=/data/builder/debuild/nginx-${NGINX_VERSION_SHORT}/debian/debuild-base/nginx-${NGINX_VERSION_SHORT}=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC" \
  && ldopt="-Wl,-z,relro" \
  \
  \
  # GeoIP2 module
  && cd "${tempDir}" \
  && git clone -b ${GEOIP2VER} https://github.com/leev/ngx_http_geoip2_module.git \
  && apt-get install --no-install-recommends --no-install-suggests -y \
    libmaxminddb0 \
    libmaxminddb-dev \
  && cd "${tempDir}/nginx-${NGINX_VERSION_SHORT}" \
  && ./configure ${buildArgs} --with-cc-opt="${ccopt}" --with-ld-opt="${ldopt}" --add-dynamic-module="${tempDir}/ngx_http_geoip2_module" \
  && make modules \
  && mv objs/ngx_http_geoip2_module.so /etc/nginx/modules \
  && apt-get remove --purge --auto-remove -y \
    libmaxminddb-dev \
  \
  \
  # HeadersMore module
  && cd "${tempDir}" \
  && git clone -b ${HEADERSMOREVER} https://github.com/openresty/headers-more-nginx-module.git \
  && cd "${tempDir}/nginx-${NGINX_VERSION_SHORT}" \
  && ./configure ${buildArgs} --with-cc-opt="${ccopt}" --with-ld-opt="${ldopt}" --add-dynamic-module="${tempDir}/headers-more-nginx-module" \
  && make modules \
  && mv objs/ngx_http_headers_more_filter_module.so /etc/nginx/modules \
  \
  \
  # Lua module
  && cd "${tempDir}" \
  && git clone -b ${LUAJIT2VER} https://github.com/openresty/luajit2.git \
  && git clone -b ${RESTYREDISVER} https://github.com/openresty/lua-resty-redis.git \
  && git clone -b ${LUANGINXVER} https://github.com/openresty/lua-nginx-module.git \
  && git clone -b ${NGXDEVVER} https://github.com/simplresty/ngx_devel_kit.git \
  && cd luajit2 && make install \
  && luajit -bg "${tempDir}/lua-resty-redis/lib/resty/redis.lua" "${tempDir}/redis.o" \
  && export LUAJIT_LIB="/usr/local/lib" \
  && export LUAJIT_INC="${tempDir}/luajit2/src/" \
  && ldopt="${ldopt},${tempDir}/redis.o,-rpath,$LUAJIT_LIB" \
  && cd "${tempDir}/nginx-${NGINX_VERSION_SHORT}" \
  && ./configure ${buildArgs} --with-cc-opt="${ccopt}" --with-ld-opt="${ldopt}" \
      --add-dynamic-module="${tempDir}/lua-nginx-module" \
      --add-dynamic-module="${tempDir}/ngx_devel_kit" \
  && make modules \
  && mv objs/ndk_http_module.so /etc/nginx/modules \
  && mv objs/ngx_http_lua_module.so /etc/nginx/modules \
  && rm -f /usr/local/lib/libluajit-5.1.a \
  # Required lua modules for Lua
  && cd "${tempDir}" \
  && git clone -b ${RESTYCOREVER} https://github.com/openresty/lua-resty-core.git \
  && git clone -b ${RESTYLRUVER} https://github.com/openresty/lua-resty-lrucache.git \
  && cd lua-resty-core && make install && ln -s /usr/local/lib/lua/resty/ /usr/local/share/lua/5.1/ && cd .. \
  && cd lua-resty-lrucache && make install && cd .. \
  \
  \
  # ModSecurity module
  && apt-get install --no-install-recommends --no-install-suggests -y \
      apt-utils \
      autoconf \
      automake \
      libcurl4 \
      libgeoip1 \
      libxml2 \
      liblmdb0 \
      libyajl2 \
      libcurl4-openssl-dev \
      libgeoip-dev \
      liblmdb-dev \
      libpcre++-dev \
      libtool \
      libxml2-dev \
      libyajl-dev \
      pkgconf \
      zlib1g-dev \
  && cd "${tempDir}" \
  && git clone -b ${MODSECCONVER} https://github.com/SpiderLabs/ModSecurity-nginx.git \
  && git clone --depth 1 -b ${MODSECLIBVER} --single-branch https://github.com/SpiderLabs/ModSecurity \
  && cd "${tempDir}/ModSecurity" \
  && git submodule init \
  && git submodule update \
  && ./build.sh \
  && ./configure \
  && make install \
  \
  && cd "${tempDir}/nginx-${NGINX_VERSION_SHORT}" \
  && ./configure ${buildArgs} --with-cc-opt="${ccopt}" --with-ld-opt="${ldopt}" --add-dynamic-module="${tempDir}/ModSecurity-nginx" \
  && make modules \
  && mv objs/ngx_http_modsecurity_module.so /etc/nginx/modules \
  \
  # Download the rules
  # Descargamos las configuraciones básicas
  && git clone --branch v3/master --progress --quiet --single-branch https://github.com/SpiderLabs/ModSecurity /tmp/ModSecurity/ \
  && mkdir -p /etc/nginx/modsec/rules.d \
  && cd /tmp/ModSecurity/ \
  && cp modsecurity.conf-recommended /etc/nginx/modsec/rules.d/modsecurity.conf \
  && cp unicode.mapping /etc/nginx/modsec/rules.d/ \
  && rm -Rf /tmp/ModSecurity/ \
  \
  # Instalación de reglas recomandadas
  && cd /tmp/ \
  && git clone --branch v3.1.1 --progress --quiet --separate-git-dir=/tmp/git https://github.com/SpiderLabs/owasp-modsecurity-crs.git /etc/nginx/modsec/crs/ \
  && rm -Rf /tmp/git \
  && cd /etc/nginx/modsec \
  && cp crs/crs-setup.conf.example crs/crs-setup.conf \
  \
  # Cleanup
  && rm -Rf /usr/local/modsecurity/include \
  && rm -f /usr/local/modsecurity/lib/libmodsecurity.a \
  && apt-get remove --purge --auto-remove -y \
      apt-utils \
      autoconf \
      automake \
      libcurl4-openssl-dev \
      libgeoip-dev \
      liblmdb-dev \
      libpcre++-dev \
      libtool \
      libxml2-dev \
      libyajl-dev \
      pkgconf \
      zlib1g-dev \
  \
  \
  # Final cleanup
  && apt-get remove --purge --auto-remove -y \
    build-essential \
    dpkg-dev \
    libpcre3-dev \
    zlib1g-dev \
    libssl-dev \
  && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
  && apt-get autoremove -y --purge && apt-get clean \
  \
  # Clean TempDir and APT
  && if [ -n "$tempDir" ]; then \
    apt-get purge -y --auto-remove \
    && rm -rf "$tempDir" /etc/apt/sources.list.d/temp.list; \
  fi

RUN set -x \
    && apt-get update && apt-get upgrade -y \
    && apt-get install --no-install-recommends --no-install-suggests -y \
       build-essential \
       gnupg2 \
    && wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add - \
    && echo "deb https://packages.sury.org/php/ buster main" > /etc/apt/sources.list.d/php.list \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y \ 
       mtr-tiny \
       iputils-ping \
       libmcrypt-dev \
       libmcrypt4 \
       php${PHP_VERSION}-fpm \
       php${PHP_VERSION}-bcmath \
       php${PHP_VERSION}-curl \
       php${PHP_VERSION}-gd \
       php${PHP_VERSION}-iconv \
       php${PHP_VERSION}-intl \
       php${PHP_VERSION}-json \
       php${PHP_VERSION}-mysql \
       php${PHP_VERSION}-mbstring \
       php${PHP_VERSION}-mcrypt \
       php${PHP_VERSION}-opcache \
       php${PHP_VERSION}-pdo \
       php${PHP_VERSION}-soap \
       php${PHP_VERSION}-xml \
       php${PHP_VERSION}-zip \
       php${PHP_VERSION}-dev \
       php-ssh2 \
       php-pear \
       php-xml \
    && pecl channel-update pecl.php.net \
    && echo "" | pecl install \
       redis \
    && echo "extension = redis.so" > /etc/php/${PHP_VERSION}/fpm/conf.d/redis.ini \
    \
    && curl https://getcomposer.org/composer-stable.phar --output /bin/composer \
    && chmod 755 /bin/composer \
    \
    # Cleanup
    && apt-get remove --purge --auto-remove -y \
       build-essential \                                               
       gnupg2 \
       libmcrypt-dev \
       php${PHP_VERSION}-dev \
       php-pear \
    && apt-get autoremove -y && apt-get clean \
    && rm -f /etc/apt/sources.list.d/php.list \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
    && apt-get install --no-install-suggests -y \
        procps \
    && rm -f /etc/apt/sources.list.d/php.list \
    && rm -rf /var/lib/apt/lists/*


STOPSIGNAL SIGQUIT
