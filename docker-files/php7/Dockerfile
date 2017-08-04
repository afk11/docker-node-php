################################################################################
# Base image
################################################################################


FROM php:7.1-fpm

################################################################################
# Work dir
################################################################################

WORKDIR /var/www/html

################################################################################
# Build instructions
################################################################################

# Install packages
RUN apt-get update && apt-get install -my \
  supervisor libgmp-dev unzip git libicu-dev libpng-dev
RUN ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h

RUN docker-php-ext-install pdo pdo_mysql gmp gd bcmath intl

# Ensure that PHP5 FPM is run as root.
# RUN sed -i "s/user = www-data/user = root/" /etc/php/fpm/pool.d/www.conf
# RUN sed -i "s/group = www-data/group = root/" /etc/php/fpm/pool.d/www.conf

# # Pass all docker environment
# RUN sed -i '/^;clear_env = no/s/^;//' /etc/php/fpm/pool.d/www.conf
#
# # Get access to FPM-ping page /ping
# RUN sed -i '/^;ping\.path/s/^;//' /etc/php/fpm/pool.d/www.conf
# # Get access to FPM_Status page /status
# RUN sed -i '/^;pm\.status_path/s/^;//' /etc/php/fpm/pool.d/www.conf

#RUN pecl install xdebug

# Add configuration files

RUN apt-get install autotools-dev automake autoconf libtool build-essential

RUN git clone https://github.com/bitcoin-core/secp256k1.git && \
    cd secp256k1 && ./autogen.sh && ./configure --enable-experimental --enable-module-ecdh --enable-module-recovery && \
    make && make install

RUN git clone https://github.com/bit-wasp/secp256k1-php.git && \
    cd secp256k1-php/secp256k1 && phpize && ./configure --with-secp256k1 && \
    make && make install

RUN apt-get install -y wget
RUN mkdir /src && cd /src/ && wget https://bitcoin.org/bin/bitcoin-core-0.14.2/bitcoin-0.14.2-x86_64-linux-gnu.tar.gz -o /src/bitcoin.tar.gz

RUN cd /src && ls && tar xvf bitcoin-0.14.2-x86_64-linux-gnu.tar.gz && cd bitcoin-0.14.2 && \
    cp include/bitcoinconsensus.h /usr/include && \
    cp lib/libbitcoinconsensus.so.0.0.0 /usr/lib && \
    ln -s /usr/lib/libbitcoinconsensus.so.0.0.0 /usr/lib/libbitcoinconsensus.so && \
    ln -s /usr/lib/libbitcoinconsensus.so.0.0.0 /usr/lib/libbitcoinconsensus.so.0 && \
    cd .. && \
    git clone -b v2.0.0 https://github.com/Bit-Wasp/bitcoinconsensus-php && \
    cd bitcoinconsensus-php/bitcoinconsensus && \
    phpize && ./configure --with-bitcoinconsensus && \
    make && make install && echo "extension=bitcoinconsensus.so" > /usr/local/etc/php/conf.d/bitcoinconsensus.ini;


# this one is only needed for tests:
RUN apt-get install -my \
    mysql-client python3 nano libzmq-dev libssl-dev libevent-dev

RUN pecl install zmq-beta
RUN docker-php-ext-install pcntl sockets
RUN pecl install event

RUN apt-get remove  --purge -my \
    autotools-dev automake autoconf libtool build-essential

RUN apt-get autoremove -my --purge

# clean
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY conf/php.ini /usr/local/etc/php/conf.d/assertions.ini
COPY conf/secp256k1.ini /usr/local/etc/php/conf.d/secp256k1.ini
COPY conf/event.ini /usr/local/etc/php/conf.d/event.ini
COPY conf/zmq.ini /usr/local/etc/php/conf.d/zmq.ini
COPY conf/php-fpm.conf /usr/local/etc/php-fpm.conf
COPY install_composer /usr/local/bin/install_composer
COPY conf/supervisord.conf /etc/supervisor/conf.d/

RUN mkdir /root/.phpnode
COPY bitcoin.ini /root/.phpnode
# Comment this if supervisor is in the way, or services are preventing the env from coming up

ENTRYPOINT /work/script.sh

################################################################################
# Ports
################################################################################

EXPOSE 9000
