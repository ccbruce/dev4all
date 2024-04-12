
FROM php:8.1-fpm-bookworm AS stage-base

# ARG DEBIAN_FRONTEND=noninteractive
ENV DEBIAN_FRONTEND noninteractive \
    LC_ALL C.UTF-8 \
    TZ Asia/Taipei 

ENV NB_USER ccbruce

RUN groupadd --gid 2001 dev \
  && groupadd --gid 1001 ccbruce \
  && useradd --uid 1000 --gid dev --shell /usr/bin/bash --create-home kag \
  && useradd --uid 1001 --gid dev --shell /usr/bin/bash --create-home ccbruce \
  && useradd --uid 1002 --gid dev --shell /usr/bin/bash --create-home mis   

ENV DEBIAN_FRONTEND noninteractive

ENV TOMCAT_VERSION 9.0.71
ENV CATALINA_HOME /usr/local/tomcat
ENV JAVA_HOME /usr/lib/jvm/java-17-openjdk-amd64
ENV CERTBOT_DNS_AUTHENTICATORS \
    cloudflare \
    cloudxns \
    digitalocean \
    dnsimple \
    dnsmadeeasy \
    gehirn \
    google \
    linode \
    luadns \
    nsone \
    ovh \
    rfc2136 \
    route53 \
    sakuracloud \
    ionos \
    bunny

# Add the OpenMediaVault repository
# COPY openmediavault.list /etc/apt/sources.list.d/
# Fix resolvconf issues with Docker
# Uncomment all Apt repositories.
RUN echo "resolvconf resolvconf/linkify-resolvconf boolean false" | debconf-set-selections \
    && apt-get update -y  \
    && apt-get install --yes --no-install-recommends \
        systemd systemd-sysv net-tools  \
        software-properties-common \
        lsb-release dctrl-tools gpg-agent gnupg gnupg2 bash \
        git make autoconf pkg-config \
        zip unzip ca-certificates bzip2\
        re2c nano htop iotop lsof cron mariadb-client redis-tools \
        busybox wget curl vim tzdata tree sshpass iputils-ping net-tools telnet gawk \
        locales openssh-server sudo \
        dnsutils supervisor dumb-init file  poppler-utils openssl \
        procps git-lfs ufw haproxy certbot\
        tar apt-utils autoconf automake build-essential \
        python3 python3-venv python3-pip python3-dev python3-setuptools python-is-python3 python3-opencv \
        python3-acme ruby-shadow tini  \
        python3-magic python3-num2words python3-odf python3-pdfminer python3-pip python3-phonenumbers \
        python3-pyldap python3-qrcode python3-renderpm python3-setuptools python3-slugify \
        python3-vobject python3-watchdog python3-xlrd python3-xlwt \
        python3-dev libldap2-dev libsasl2-dev ldap-utils tox lcov valgrind \
        xz-utils gnupg luajit busybox ghostscript imagemagick ffmpeg libgdiplus \
        fonts-noto-cjk \
        nodejs npm yarn\
        nginx  python3-certbot-nginx \
        bind9 bind9-doc bind9-utils\
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev libzip-dev \
        rustc openssh-client tmux rlwrap rsync fakeroot  less jq \
        xz-utils gnupg luajit busybox openjdk-17-jdk \
        ant patch xmlstarlet  \
        # .NET dependencies
        # libc6 \
        # libgcc1 \
        # libgssapi-krb5-2 \
        # libicu66 \
        # libssl1.1 \
        # libstdc++6 \
        # zlib1g \
    # Remove Apt cache to shrink the image size.    
    && npm install -g pnpm \
    # Remove Apt cache to shrink the image size.
    && mkdir -p /etc/backup/ \
    #&& cp -fr /etc/haproxy/* /etc/backup/haproxy \ 
    && dpkg-reconfigure locales \
    #&& sed -i 's/# deb/deb/g' /etc/apt/sources.list \
    # Add the repository to Apt sources:
    && sudo apt-get -y install python3-certbot-nginx \
    # Download the Microsoft repository GPG keys
    && apt-get clean autoclean \
    && apt-get autoremove --yes \
    && rm -rf ~/.cache/pip /var/lib/apt/lists/* /tmp/* /var/tmp/*  \
    && rm -rf /var/lib/apt/lists/* \
    && echo '*/5 * * * * php -f /var/www/html/cron.php' > /var/spool/cron/crontabs/www-data \
    && chown www-data:www-data -R /var/www 

RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash \
    && export NVM_DIR="$HOME/.nvm" \
	&& [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"   \
	# This loads nvm
	&& [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  \
	&& nvm install 16 \
	&& nvm install 20 \
	&& git config --global http.sslverify "false" 

RUN wget https://github.com/PowerShell/PowerShell/releases/download/v7.4.1/powershell_7.4.1-1.deb_amd64.deb \
    # Register the Microsoft repository GPG keys
    && dpkg -i powershell_7.4.1-1.deb_amd64.deb \
    # Delete the Microsoft repository GPG keys file
    && rm powershell_7.4.1-1.deb_amd64.deb \
    # Update the list of packages after we added packages.microsoft.com
    && apt-get update \
    ###################################
    # Install PowerShell
    && apt-get install -y powershell  \    
    && apt-get clean autoclean \
    && apt-get autoremove --yes \
    && rm -rf ~/.cache/pip /var/lib/apt/lists/* /tmp/* /var/tmp/*  \
    && rm -rf /var/lib/apt/lists/* 

# Composer
RUN curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php && \
    curl -sS https://composer.github.io/installer.sig -o /tmp/composer-setup.sig && \
    php -r "if (hash_file('sha384', '/tmp/composer-setup.php') !== trim(file_get_contents('/tmp/composer-setup.sig'))) { echo 'Composer installation failed, invalid hash'; exit(1); }" && \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
    rm /tmp/composer-setup.php /tmp/composer-setup.sig

RUN pecl install xdebug-3.1.5
RUN docker-php-ext-enable xdebug \
    && docker-php-ext-install zip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd

# Get Go
ENV GOLANG_VERSION 1.22.2
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA256 000a5b1fca4f75895f78befeb2eecf10bfff3c428597f3f1e69133b63b911b02

RUN curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
    # && echo "$GOLANG_DOWNLOAD_SHA256  golang.tar.gz" | sha256sum -c - \ #skip check
    && tar -C /usr/local -xzf golang.tar.gz \
    && rm golang.tar.gz

# Linux and Glibc build dependencies and emulator
RUN apt-get update && apt-get install -y  --no-install-recommends \
        bison gawk make python3 \
        gcc gcc-multilib \
        gettext texinfo \
        qemu-user \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
# Cross compilers (install recommended packages to get cross libc-dev)
RUN apt-get update && apt-get install -y \
        gcc-aarch64-linux-gnu       gcc-arm-linux-gnueabi     \
        gcc-mips-linux-gnu          gcc-mips64-linux-gnuabi64 \
        gcc-mips64el-linux-gnuabi64 gcc-mipsel-linux-gnu      \
        gcc-powerpc-linux-gnu       gcc-powerpc64-linux-gnu   \
        gcc-powerpc64le-linux-gnu   gcc-riscv64-linux-gnu     \
        gcc-s390x-linux-gnu         gcc-sparc64-linux-gnu     \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Only for loong64, getting tools of qemu-user and gcc-cross-compiler
ENV LOONG64_BASE_URL https://github.com/loongson/build-tools/releases/download/2022.09.06
ENV LOONG64_GCC  loongarch64-clfs-6.3-cross-tools-gcc-glibc.tar.xz
ENV LOONG64_QEMU qemu-loongarch64
ENV LOONG64_GCC_DOWNLOAD_URL  $LOONG64_BASE_URL/$LOONG64_GCC
ENV LOONG64_QEMU_DOWNLOAD_URL $LOONG64_BASE_URL/$LOONG64_QEMU

RUN apt-get update && apt-get install xz-utils -y && mkdir /loong64 && cd /loong64 \
    && curl -fsSL "$LOONG64_QEMU_DOWNLOAD_URL" -o /usr/bin/"$LOONG64_QEMU" \
    && chmod +x /usr/bin/"$LOONG64_QEMU" \
    && curl -fsSL "$LOONG64_GCC_DOWNLOAD_URL" -o "$LOONG64_GCC" \
    && tar xf "$LOONG64_GCC" -C /usr/local/ \
    && ln -s /usr/local/cross-tools/bin/loongarch64-unknown-linux-gnu-gcc /usr/bin/loongarch64-linux-gnu-gcc \
    && rm -rf /loong64

# Install and configure Tomcat.
RUN mkdir $CATALINA_HOME
RUN wget https://archive.apache.org/dist/tomcat/tomcat-9/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz -O /tmp/tomcat.tar.gz
RUN cd /tmp && tar xvfz tomcat.tar.gz
RUN cp -Rv /tmp/apache-tomcat-${TOMCAT_VERSION}/* $CATALINA_HOME
RUN rm -rf /tmp/apache-tomcat-${TOMCAT_VERSION}
RUN rm -rf /tmp/tomcat.tar.gz

# Retrieve .NET Runtime
RUN dotnet_version=7.0.18 \
    && curl -fSL --output dotnet.tar.gz https://dotnetcli.azureedge.net/dotnet/Runtime/$dotnet_version/dotnet-runtime-$dotnet_version-linux-x64.tar.gz \
    && dotnet_sha512='9d2aaf11e798d8dbfa74a93cfc53c6bb631cfb041b5dc55c208f980f61808e872dfa9880c7d9d4b42aad934e5350c9e8f327664909054fa0109636158701b4b8' \
    && echo "$dotnet_sha512  dotnet.tar.gz" | sha512sum -c - \
    && mkdir -p /dotnet \
    && tar -oxzf dotnet.tar.gz -C /dotnet \
    && rm dotnet.tar.gz

COPY docker-entrypoint.sh /opt/startup/docker-entrypoint.sh

RUN  echo 'export PATH="/workspace/bin:$PATH"' >> /home/ccbruce/.bashrc \
    && mkdir -p /opt/startup /workspace/app /var/workspace /var/run/sshd /var/my-docker /var/repository && chmod 777 /workspace\
    && echo 'ccbruce ALL=(ALL:ALL) ALL' >> /etc/sudoers \
    && echo 'kag ALL=(ALL:ALL) ALL' >> /etc/sudoers \
    && echo 'mis ALL=(ALL:ALL) ALL' >> /etc/sudoers \
    && echo 'ccbruce:mithra35' | chpasswd \
    && echo 'kag:mithra35' | chpasswd \
    && echo 'mis:mithra35' | chpasswd \
    && sed -i "s/#Port.*/Port 22/" /etc/ssh/sshd_config \
    && sed -i "s/#PasswordAuthentication.*/PasswordAuthentication no/" /etc/ssh/sshd_config \
    && sed -i "s/PasswordAuthentication.*/PasswordAuthentication no/" /etc/ssh/sshd_config \
    && sed -i "s/UsePAM.*yes/UsePAM no/" /etc/ssh/sshd_config \
    && sed -i "s/#AuthorizedKeysFile/AuthorizedKeysFile/" /etc/ssh/sshd_config \
    && sed -i "s/#UsePAM.*yes/UsePAM no/" /etc/ssh/sshd_config \
    && sed -i "s/#StrictModes yes/StrictModes no/" /etc/ssh/sshd_config \
    && sed -i "s/#PubkeyAuthentication.*/PubkeyAuthentication yes/" /etc/ssh/sshd_config \
    && echo "KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1" >> /etc/ssh/sshd_config \
    && echo "*****SSH login fix. Otherwise user is kicked off after login*****" \
    #&& sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd \
    && mkdir -p /var/log/supervisor \
    && touch /opt/startup/monitor.log \
    && touch /var/log/supervisor/supervisord.log \
    && chmod 666 /var/log/supervisor/supervisord.log \
    && chmod +x /opt/startup/docker-entrypoint.sh \
    && rm -rf ~/.cache/pip /var/lib/apt/lists/* /tmp/* /var/tmp/*

EXPOSE 53 22 80 443 8080 8888 3000

ENTRYPOINT ["/usr/bin/dumb-init", "--", "/opt/startup/docker-entrypoint.sh"]
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]