ARG nginx_version=1.19.0

FROM nginx:${nginx_version} AS build

SHELL ["/bin/bash", "-c"]

# Install packages
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-suggests \
    autoconf \
    apt-utils \
    automake \
    build-essential \
    ca-certificates \
    curl \
    dnsutils \
    git \
    iputils-ping \
    libcurl4-openssl-dev \
    libexpat1-dev \
    libpam0g-dev \
    libpcre3 \
    libpcre3-dev \
    libssl-dev \
    libtool \
    libxml2 \
    libxslt1-dev \
    libxslt1.1 \
    libyajl-dev \
    net-tools \
    pkgconf \
    procps \
    rsync \
    tcpdump \
    unzip \
    uuid-dev \
    vim-tiny \
    wget \
    zlib1g-dev

# Define user and group

RUN set -x \
    && addgroup --system --gid 101 nginx \
    && adduser --system --disabled-login --ingroup nginx --no-create-home --gecos "nginx user" --shell /bin/false --uid 101 nginx

# Fetch ngx_pagespeed

ARG pagespeed_version=1.13.35.2-stable

RUN set -x \
    && cd /usr/local/src \
    && wget https://github.com/apache/incubator-pagespeed-ngx/archive/v${pagespeed_version}.zip \
    && unzip v${pagespeed_version}.zip \
    && nps_dir=$(find . -name "*pagespeed-ngx-${pagespeed_version}" -type d) \
    && cd "$nps_dir" \
    && NPS_RELEASE_NUMBER=${pagespeed_version/beta/} \
    && NPS_RELEASE_NUMBER=${pagespeed_version/stable/} \
    && psol_url=https://github.com/pagespeed/ngx_pagespeed/archive/v${pagespeed_version}.tar.gz \
    && [ -e scripts/format_binary_url.sh ] && psol_url=$(scripts/format_binary_url.sh PSOL_BINARY_URL) \
    && wget ${psol_url} \
    && tar -xzvf $(basename ${psol_url})

# Fetch modsecurity

ARG modsecurity_version=v3.0.4

RUN set -x \
    && git clone --depth 1 -b ${modsecurity_version} https://github.com/SpiderLabs/ModSecurity.git /usr/local/src/modsecurity \
    && cd /usr/local/src/modsecurity \
    && mkdir /etc/nginx/modsec \
    && sed -i "s/SecRuleEngine DetectionOnly/SecruleEngine On/g" modsecurity.conf-recommended \
    && cp modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf \ 
    && git submodule init \
    && git submodule update \
    && ./build.sh \
    && ./configure --disable-mlogc \
    && make \
    && make install

# Fetch Nginx connector

ARG modsecurity_connector_version=1.0.1

RUN set -x \
    && mkdir /tmp/modsecurityconnector \
    && cd /tmp/modsecurityconnector \
    && git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
    
# Fetch core rule set for modsecurity

ARG owasp_version=3.2.0

RUN set -x \
    && modsecurity_dir="/usr/local/etc/modsecurity" \
    && mkdir -p ${modsecurity_dir} \
    && cd ${modsecurity_dir} \
    && curl -fSL "https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v${owasp_version}.tar.gz" | tar -xvzf - \
    && mv owasp-modsecurity-crs-${owasp_version} /usr/local/owasp-modsecurity-crs \
    && cd /usr/local/owasp-modsecurity-crs \
    && cp crs-setup.conf.example crs-setup.conf \
    && cd -

# Install Nginx

ARG nginx_version=1.19.0
ARG njs_version=0.4.1
ARG pagespeed_version=1.13.35.2-stable

RUN set -x \
    && mkdir /tmp/nginx \
    && cd /tmp/nginx \
    && wget http://nginx.org/download/nginx-${nginx_version}.tar.gz \
    && tar -xvzf nginx-${nginx_version}.tar.gz \
    && cd nginx-${nginx_version}/ \
    && ./configure \
    --add-dynamic-module=/tmp/modsecurityconnector/ModSecurity-nginx \
    --add-module=/usr/local/src/incubator-pagespeed-ngx-${pagespeed_version} \
    --sbin-path=/usr/sbin \
    --modules-path=/usr/lib/nginx \
    --with-http_ssl_module \
    --with-http_gzip_static_module \
    --with-file-aio \
    --with-http_v2_module \
    --with-http_realip_module \
    --without-http_autoindex_module \
    --without-http_browser_module \
    --without-http_userid_module \
    --without-mail_pop3_module \
    --without-mail_imap_module \
    --without-mail_smtp_module \
    --without-http_split_clients_module \
    --without-http_uwsgi_module \
    --without-http_scgi_module \
    --without-http_upstream_ip_hash_module \
    --with-http_sub_module \
    --with-http_gunzip_module \
    --with-http_secure_link_module \
    --with-threads \
    --with-stream \
    --with-compat \
    --with-stream_ssl_module \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --pid-path=/var/run/nginx.pid \
    --with-cc-opt="-fPIC -I /usr/include/apr-1" \
#    --with-ld-opt="-luuid -lapr-1 -laprutil-1 -licudata -licuuc -L /tmp/modpagespeed-${pagespeed_version}/usr/lib -lpng12 -lturbojpeg -ljpeg" \
    && make modules \
    && cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules \
    && make \
    && make install

# Clean our nginx directory
RUN set -x \
    && cd /etc/nginx \
    && rm -rf client_body_temp conf.d fast* html koi* mime* nginx* proxy_temp scgi* uws* win-utf logs sites-*

# Copy our prepared files
COPY ./files/ /etc/nginx/
COPY ./modsec/main.conf /etc/nginx/modsec/main.conf

RUN set -x \
    && apt-get update \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && ldconfig -v \
    && ls /etc/nginx/modules/*.so | grep -v debug | xargs -I{} sh -c 'echo "load_module {};" | tee -a  /etc/nginx/modules/all.conf' \
    && ln -sf /dev/stdout /var/log/modsec_audit.log \
    && touch /var/run/nginx.pid \
    && mkdir -p /var/cache/nginx \
    && mkdir -p /var/www/html/example.com/cache \
    && mkdir -p /var/www/html/example.com/html \
    && mkdir -p /var/www/html/example.com/ps_cache \
    && mkdir -p /var/www/html/example.com/logs \
    && chown -R nginx:nginx /etc/nginx /var/log/nginx /var/cache/nginx /var/run/nginx.pid /var/log/modsec_audit.log /var/cache/cache-heater \
    && echo 1 > /var/run/nginx.pid \
    && nginx -s reload

EXPOSE 80 443

WORKDIR /etc/nginx
