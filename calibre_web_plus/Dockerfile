FROM ghcr.io/linuxserver/baseimage-ubuntu:arm32v7-focal
#arm32v7/ubuntu:latest

# set version label
ARG BUILD_DATE
ARG VERSION
ARG BUILD_VERSION
ARG BUILD_ARCH
ARG BASHIO_VERSION=0.13.1
# Add env
ENV LANG C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive 
ENV TZ=Europe/Rome

# Setup base
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get clean
RUN apt-get update 
RUN apt-get install -y curl
RUN apt-get install -y jq

#Install python3
RUN apt install -y python3

#Install pip3
RUN apt-get update
RUN apt install -y python3-pip

RUN apt-get install -y libxml2-dev
RUN apt-get install -y libxslt-dev
RUN apt-get install -y libffi-dev
RUN apt-get install -y rustup

RUN pip install --upgrade pip
RUN pip install --upgrade setuptools


# Install calibre web
RUN pip3 install calibreweb

# ADD Calibre for ebook conversion
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata
RUN apt-get install -y --no-install-recommends calibre

#NGINX
RUN apt-get install -y nginx

#BASHIO
RUN apt-get install -y bash
RUN apt-get install -y ca-certificates 
RUN mkdir -p /tmp/bashio && curl -L -s https://github.com/hassio-addons/bashio/archive/v${BASHIO_VERSION}.tar.gz | tar -xzf - --strip 1 -C /tmp/bashio
RUN mv /tmp/bashio/lib /usr/lib/bashio && ln -s /usr/lib/bashio/bashio /usr/bin/bashio
RUN rm -rf /tmp/bashio

# Install calibre web Optional features
RUN pip3 install calibreweb[metadata]
# RUN pip3 install calibreweb[gdrive]
# RUN pip3 install calibreweb[gmail]
# RUN pip3 install calibreweb[goodreads]
# RUN pip3 install calibreweb[ldap]
# RUN pip3 install calibreweb[oauth]
# RUN pip3 install calibreweb[comics]
# RUN pip3 install calibreweb[kobo]


#Clean up
RUN apt-get -y autoremove
RUN rm -rf /tmp/* 
RUN rm -rf /var/lib/apt/lists/*
RUN rm -rf /var/tmp/*


## Copy data
COPY nginx.conf /etc/
COPY run.sh /

RUN chmod a+x /run.sh

CMD [ "/run.sh" ]
