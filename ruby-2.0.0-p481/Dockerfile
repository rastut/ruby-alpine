FROM alpine:3.4

# Skip installing gem documentation
#
RUN mkdir -p /usr/local/etc \
  && { \
    echo 'install: --no-document'; \
    echo 'update: --no-document'; \
  } >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.0
ENV RUBY_VERSION 2.0.0-p481
ENV RUBYGEMS_VERSION 2.6.4
COPY io.c.patch /

RUN apk add --no-cache --virtual .ruby-builddeps autoconf bison bzip2 \
    bzip2-dev ca-certificates coreutils curl gcc gdbm-dev glib-dev libc-dev \
    libffi-dev libxml2-dev libxslt-dev linux-headers make ncurses-dev \
    openssl-dev procps readline-dev ruby yaml-dev zlib-dev \
    && curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
    && apk add geos-dev geos --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted \
    # Set up build, and tuneup some files
    #
    && mkdir -p /usr/src \
    && tar -xzf ruby.tar.gz -C /usr/src \
    && mv "/usr/src/ruby-$RUBY_VERSION" /usr/src/ruby \
    && rm ruby.tar.gz  \
    && cd /usr/src/ruby  \
    && { echo '#define ENABLE_PATH_CHECK 0'; echo; cat file.c; } > file.c.new && mv file.c.new file.c \
    # Configuring, build and install
    #
    && autoconf \
    && ac_cv_func_isnan=yes ac_cv_func_isinf=yes \
      ./configure --disable-install-doc \
    && patch /usr/src/ruby/io.c /io.c.patch \
    && make -j"$(getconf _NPROCESSORS_ONLN)" \
    && make install \
    && runDeps="$( \
      scanelf --needed --nobanner --recursive /usr/local \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
      )" \
    # Cleanup pkg and src
    #
    && apk add --virtual .ruby-rundeps $runDeps bzip2 ca-certificates curl libffi-dev openssl-dev \
      yaml-dev procps zlib-dev \
    && apk del .ruby-builddeps \
    && gem update --system $RUBYGEMS_VERSION \
    && rm -r /usr/src/ruby

ENV BUNDLER_VERSION 1.12.5

RUN gem install bundler --version "$BUNDLER_VERSION"

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
  BUNDLE_BIN="$GEM_HOME/bin" \
  BUNDLE_SILENCE_ROOT_WARNING=1 \
  BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $BUNDLE_BIN:$PATH
RUN mkdir -p "$GEM_HOME" "$BUNDLE_BIN" \
  && chmod 777 "$GEM_HOME" "$BUNDLE_BIN"
