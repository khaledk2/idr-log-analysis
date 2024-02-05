FROM fluentd:v1.16.0-debian-1.0
USER root
RUN  apt-get update -y
RUN apt-get install -y \
lzma \
build-essential \
automake \
autoconf \
libtool \
libreadline-dev \
libc6-dev

RUN apt-get install -y \
libgeoip-dev \
zlib1g-dev \
libssl-dev \
ruby-dev

RUN apt-get -qq -y install curl
RUN apt-get install geoip-bin
RUN apt-get install geoip-database

# Install gems
RUN gem install bundler rubygems-bundler
RUN gem regenerate_binstubs
RUN gem install fluent-plugin-geoip
RUN gem install elasticsearch -v 7.17.10
RUN gem install fluent-plugin-elasticsearch --no-document --version 5.4.1
RUN gem install fluent-plugin-rewrite-tag-filter
RUN gem install fluent-plugin-multi-format-parser
RUN gem install fluent-plugin-slack
RUN gem install fluent-plugin-flowcounter

USER fluent
