# 使用 Debian Bookworm 作为基础镜像
FROM arm64v8/debian:bookworm-slim

# 镜像维护者信息
LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

# 设置环境变量
ENV NGINX_VERSION=1.27.0
ENV NJS_VERSION=0.8.4
ENV NJS_RELEASE=2~bookworm
ENV PKG_RELEASE=2~bookworm

# 安装必要依赖并配置 NGINX
RUN set -x \
    && groupadd --system --gid 101 nginx \
    && useradd --system --gid nginx --no-create-home --home /nonexistent --comment "nginx user" --shell /bin/false --uid 101 nginx \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates \
    && NGINX_GPGKEYS="573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 8540A6F18833A80E9C1653A42FD21310B49F6B46 9E9BE90EACBCDE69FE9B204CBCDCD8A38D88A2B3" \
    && NGINX_GPGKEY_PATH=/etc/apt/keyrings/nginx-archive-keyring.gpg \
    && export GNUPGHOME="$(mktemp -d)" \
    && found='' \
    && for NGINX_GPGKEY in $NGINX_GPGKEYS; do \
         for server in hkp://keyserver.ubuntu.com:80 pgp.mit.edu; do \
            echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
            gpg1 --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
         done; \
         test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
       done; \
    gpg1 --export "$NGINX_GPGKEYS" > "$NGINX_GPGKEY_PATH" ; \
    rm -rf "$GNUPGHOME"; \
    apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/* \
    && dpkgArch="$(dpkg --print-architecture)" \
    && nginxPackages=" \
         nginx=${NGINX_VERSION}-${PKG_RELEASE} \
         nginx-module-xslt=${NGINX_VERSION}-${PKG_RELEASE} \
         nginx-module-geoip=${NGINX_VERSION}-${PKG_RELEASE} \
         nginx-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE} \
         nginx-module-njs=${NGINX_VERSION}+${NJS_VERSION}-${NJS_RELEASE} \
       " \
    && case "$dpkgArch" in \
         amd64|arm64) \
            echo "deb [signed-by=$NGINX_GPGKEY_PATH] https://nginx.org/packages/mainline/debian/ bookworm nginx" >> /etc/apt/sources.list.d/nginx.list \
            && apt-get update \
            ;; \
         *) \
            echo "deb-src [signed-by=$NGINX_GPGKEY_PATH] https://nginx.org/packages/mainline/debian/ bookworm nginx" >> /etc/apt/sources.list.d/nginx.list \
            && tempDir="$(mktemp -d)" \
            && chmod 777 "$tempDir" \
            && savedAptMark="$(apt-mark showmanual)" \
            && apt-get update \
            && apt-get build-dep -y $nginxPackages \
            && ( \
                 cd "$tempDir" \
                 && DEB_BUILD_OPTIONS="nocheck parallel=$(nproc)" \
                    apt-get source --compile $nginxPackages \
               ) \
            && apt-mark showmanual | xargs apt-mark auto > /dev/null \
            && { [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; } \
            && ls -lAFh "$tempDir" \
            && ( cd "$tempDir" && dpkg-scanpackages . > Packages ) \
            && grep '^Package: ' "$tempDir/Packages" \
            && echo "deb [ trusted=yes ] file://$tempDir ./" > /etc/apt/sources.list.d/temp.list \
            && apt-get -o Acquire::GzipIndexes=false update \
            ;; \
       esac \
    && apt-get install --no-install-recommends --no-install-suggests -y \
         $nginxPackages \
         gettext-base \
         curl \
    && apt-get remove --purge --auto-remove -y && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
    && if [ -n "$tempDir" ]; then \
         apt-get purge -y --auto-remove \
         && rm -rf "$tempDir" /etc/apt/sources.list.d/temp.list; \
       fi \
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && mkdir /docker-entrypoint.d

# 复制配置和脚本
COPY docker-entrypoint.sh /
COPY 10-listen-on-ipv6-by-default.sh /docker-entrypoint.d/
COPY 15-local-resolvers.envsh /docker-entrypoint.d/
COPY 20-envsubst-on-templates.sh /docker-entrypoint.d/
COPY 30-tune-worker-processes.sh /docker-entrypoint.d/
COPY emby115.conf /etc/nginx/conf.d/

# 删除默认配置文件
RUN rm /etc/nginx/conf.d/default.conf

# 设置入口点和默认命令
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]

# 暴露端口和停止信号
EXPOSE 80/tcp
STOPSIGNAL SIGQUIT
