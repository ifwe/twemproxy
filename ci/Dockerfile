# Dockerfile to build nutcracker
# ARGS: none
# Also see travis.sh
FROM centos:7

ENV LAST_MODIFIED_DATE 2017-12-08

# socat Allow tests to open log files
# which (used below)
# python-setuptools for pip
RUN yum install -y \
        tar git gcc make tcl \
        autoconf automake libtool wget \
        memcached \
		socat \
		which \
		python-setuptools && \
	yum clean all

# Install nosetest dependencies
RUN easy_install pip && \
        pip install nose && \
        pip install 'git+https://github.com/andymccurdy/redis-py.git@2.10.6' && \
        pip install 'git+https://github.com/idning/python-memcached.git#egg=memcache'
# I can't install redis or redis-server in centos:7 (didn't add package), adding to centos 6
# Install redis and redis sentinel, needed for unit tests
# RUN yum install -y redis redis-sentinel

ARG REDIS_VER=3.2.11
RUN wget https://github.com/antirez/redis/archive/$REDIS_VER.tar.gz && \
	tar zxvf $REDIS_VER.tar.gz && \
	pushd redis-$REDIS_VER && \
		make install && \
	popd && \
	rm -r redis-*

# This will build and twemproxy 0.4.1 (with sflow)
# Annoyingly, can't add multiple directories at once.
ADD conf /usr/src/twemproxy/conf
ADD contrib /usr/src/twemproxy/contrib
ADD man /usr/src/twemproxy/man
ADD m4 /usr/src/twemproxy/m4
ADD notes /usr/src/twemproxy/notes
ADD scripts /usr/src/twemproxy/scripts
ADD src /usr/src/twemproxy/src
ADD ChangeLog configure.ac Makefile.am LICENSE NOTICE README.md /usr/src/twemproxy/

WORKDIR /usr/src/twemproxy


ADD ci/build-nutcracker.sh /usr/local/bin/build-nutcracker.sh
RUN /usr/local/bin/build-nutcracker.sh

# Add the tests after adding source files, which makes it easy to quickly test changes to unit tests.
ADD tests /usr/src/twemproxy/tests

# Not installing redis utilities, since we're not running those tests.
RUN mkdir tests/_binaries -p

RUN ln -nsf $PWD/src/nutcracker tests/_binaries/nutcracker && \
    cp `which redis-server` tests/_binaries/redis-server && \
    cp `which redis-server` tests/_binaries/redis-sentinel && \
    cp `which memcached` tests/_binaries/memcached && \
    cp `which redis-cli` tests/_binaries/redis-cli

WORKDIR /usr/src/twemproxy/tests

# Allow tests to open log files
RUN chmod -R a+w log/
RUN cat /etc/passwd
RUN chown -R daemon:daemon /usr/src/twemproxy
USER daemon
