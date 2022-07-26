FROM ghcr.io/linuxserver/baseimage-ubuntu:arm32v7-focal
#arm32v7/ubuntu:latest

# set version label
ARG BUILD_DATE
ARG VERSION
ARG BUILD_VERSION
ARG BUILD_ARCH
ARG BASHIO_VERSION=0.13.1
ARG CALIBREWEB_RELEASE

# Add env
ENV LANG C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive 
ENV TZ=Europe/Rome


RUN \
  echo "**** install build packages ****" && \
  apt-get update && \
  apt-get install -y \
    git \
    libldap2-dev \
    libsasl2-dev \
    python3-pip && \
  echo "**** install runtime packages ****" && \
  apt-get install -y \
    imagemagick \
    libnss3 \
    libxcomposite1 \
    libxi6 \
    libxslt1.1 \
    libldap-2.4-2 \
    libsasl2-2 \
    libxrandr2 \
    python3-minimal \
    python3-pkg-resources \
    unrar && \
  echo "**** install calibre-web ****" && \
  if [ -z ${CALIBREWEB_RELEASE+x} ]; then \
    CALIBREWEB_RELEASE=$(curl -sX GET "https://api.github.com/repos/janeczku/calibre-web/releases/latest" \
      | awk '/tag_name/{print $4;exit}' FS='[""]'); \
  fi && \
  curl -o \
    /tmp/calibre-web.tar.gz -L \
    https://github.com/janeczku/calibre-web/archive/${CALIBREWEB_RELEASE}.tar.gz && \
  mkdir -p \
    /app/calibre-web && \
  tar xf \
    /tmp/calibre-web.tar.gz -C \
    /app/calibre-web --strip-components=1 && \
  cd /app/calibre-web && \
  pip3 install --no-cache-dir -U \
    pip && \
  pip install --no-cache-dir -U --ignore-installed --find-links https://wheel-index.linuxserver.io/ubuntu/ -r \
    requirements.txt -r \
    optional-requirements.txt

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

# Set S6 wait time
ENV S6_CMD_WAIT_FOR_SERVICES=1 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_SERVICES_GRACETIME=0

# Allow UID and GID setting
RUN if [ -f /etc/cont-init.d/02-tamper-check ]; then rm /etc/cont-init.d/02-tamper-check; fi \
    && sed -i 's/bash/bashio/g' /etc/cont-init.d/10-adduser \
    && sed -i 's/{PUID:-911}/(bashio::config "PUID")/g' /etc/cont-init.d/10-adduser \
    && sed -i 's/{PGID:-911}/(bashio::config "PGID")/g' /etc/cont-init.d/10-adduser 

#Clean up.
RUN \
  echo "**** cleanup ****" && \
  apt-get -y purge \
    git \
    libldap2-dev \
    libsasl2-dev \
    python3-pip && \
  apt-get -y autoremove && \
  rm -rf \
    /tmp/* \
    /var/lib/apt/lists/* \
    /var/tmp/* \
    /root/.cache

# Corrects permissions for s6 v3
RUN if [ -d /etc/cont-init.d ]; then chmod -R 755 /etc/cont-init.d; fi && \
    if [ -d /etc/services.d ]; then chmod -R 755 /etc/services.d; fi && \
    if [ -f /entrypoint.sh ]; then chmod 755 /entrypoint.sh; fi


## Copy data
COPY nginx.conf /etc/
COPY run.sh /

RUN chmod a+x /run.sh

CMD [ "/run.sh" ]
