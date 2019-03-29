# Zend Server
#
# Version 8.5.12+b817

FROM ubuntu:14.04

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key 799058698E65316A2E7A4FF42EAE1437F7D2C623 
RUN echo "deb http://repos.zend.com/zend-server/8.5.12/deb_apache2.4 server non-free" >> /etc/apt/sources.list.d/zend-server.list
RUN apt-get update && apt-get install -y curl unzip libmysqlclient18 git wget gcc make python-software-properties autoconf zend-server-php-5.6=8.5.12+b817 && /usr/local/zend/bin/zendctl.sh stop

COPY ./zend.lic /etc/

#Make apache drop the HTTP_PROXY header to fix CVE-2016-5385, CVE-2016-5387
COPY ./drop-http-proxy-header.conf /etc/apache2/conf-available
RUN  /usr/sbin/a2enconf drop-http-proxy-header
RUN  /usr/sbin/a2enmod headers

# "zs-init" is a standard Zend Server cloud initialization package.
# It has minor tweaks for use within Docker which can be found at https://github.com/zendtech/zs-init/tree/docker
ENV ZS_INIT_VERSION 0.3
ENV ZS_INIT_SHA256 e8d441d8503808e9fc0fafc762b2cb80d4a6e68b94fede0fe41efdeac10800cb
RUN curl -fSL -o zs-init.tar.gz "http://repos.zend.com/zs-init/zs-init-docker-${ZS_INIT_VERSION}.tar.gz" \
    && echo "${ZS_INIT_SHA256} *zs-init.tar.gz" | sha256sum -c - \
    && mkdir /usr/local/zs-init \
    && tar xzf zs-init.tar.gz --strip-components=1 -C /usr/local/zs-init \
    && rm zs-init.tar.gz

WORKDIR /usr/local/zs-init
RUN /usr/local/zend/bin/php -r "readfile('https://getcomposer.org/installer');" | /usr/local/zend/bin/php
RUN /usr/local/zend/bin/php composer.phar update

COPY ./scripts /usr/local/bin
COPY ./Zray /usr/local/zend/var/plugins/

#Compile OCI 8-2.0.12
COPY ./oci/ /tmp/

WORKDIR /tmp
RUN unzip instantclient-basic-linux.x64-11.2.0.4.0.zip -d /opt
RUN unzip instantclient-sdk-linux.x64-11.2.0.4.0.zip -d /opt
RUN tar -xzf oci8-2.0.12.tgz

WORKDIR /opt/instantclient_11_2
RUN ln -s libclntsh.so.11.1 libclntsh.so
RUN ln -s libocci.so.11.1 libocci.so

WORKDIR /tmp/oci8-2.0.12
RUN /usr/local/zend/bin/phpize
RUN ./configure --with-oci8=instantclient,/opt/instantclient_11_2 --with-php-config=/usr/local/zend/bin/php-config
RUN make install

EXPOSE 80
EXPOSE 443
EXPOSE 10081
EXPOSE 10082

WORKDIR /var/www

COPY vhost.conf /etc/apache2/sites-enabled/vhost.conf

CMD ["/usr/local/bin/run"]

