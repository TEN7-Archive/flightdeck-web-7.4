FROM alpine:3.13
MAINTAINER tess@ten7.com

# Update the package list and install Ansible.
RUN apk -U upgrade &&\
    apk add --update --no-cache ansible && \
    rm -rf /tmp/* \
           /var/cache/apk/* \
           /usr/lib/python3.6/site-packages/ansible/modules/network/ \
           /usr/lib/python3.6/site-packages/ansible/modules/cloud/ \
           /usr/lib/python3.6/site-packages/ansible/modules/windows/

# Copy the Ansible configuration files
COPY ansible-hosts /etc/ansible/hosts
COPY ansible.cfg /etc/ansible/ansible.cfg
COPY ansible /ansible

# Run the build.
RUN ansible-galaxy install -fr /ansible/requirements.yml && \
    ansible-playbook -i /etc/ansible/hosts /ansible/build.yml

# Fix iconv extension
# See https://github.com/docker-library/php/issues/240
ENV PHP_VER="7.4.15"
ARG BUILD_PACKAGES="wget build-base php7-dev"

RUN apk add --no-cache --virtual .php-build-dependencies $BUILD_PACKAGES && \
    apk add --no-cache --repository https://dl-cdn.alpinelinux.org/alpine/edge/testing/ gnu-libiconv-dev && \
    (mv /usr/bin/gnu-iconv /usr/bin/iconv; mv /usr/include/gnu-libiconv/*.h /usr/include; rm -rf /usr/include/gnu-libiconv) && \
    mkdir -p /opt && \
    cd /opt && \
    wget https://secure.php.net/distributions/php-$PHP_VER.tar.gz && \
    tar xzf php-$PHP_VER.tar.gz && \
    cd php-$PHP_VER/ext/iconv && \
    phpize && \
    ./configure --with-iconv=/usr && \
    make && \
    make install && \
    mkdir -p /etc/php7/conf.d && \
    apk del .php-build-dependencies && \
    rm -rf /opt/*
# END Fix iconv extension

# Allow Apache to allocate ports as non-root.
RUN setcap cap_net_bind_service=+ep /usr/sbin/httpd

# Configure the runtime environment of the container.
EXPOSE 80
WORKDIR /var/www

# Switch to the service account.
USER apache

# Set the entrypoint and default command.
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["docker-cmd.sh", "/usr/sbin/httpd", "-D", "FOREGROUND", "-f", "/etc/apache2/httpd.conf"]
