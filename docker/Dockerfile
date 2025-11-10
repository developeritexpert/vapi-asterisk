# Base image
FROM debian:12

LABEL maintainer="rahul <rahul@local.dev>"
LABEL description="Asterisk 18.22.0 with PJSIP built from source on Debian 12"

ENV DEBIAN_FRONTEND=noninteractive

# Update and install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      build-essential git wget curl subversion \
      libxml2-dev libncurses5-dev uuid-dev \
      libjansson-dev libssl-dev libsqlite3-dev \
      libedit-dev libcurl4-openssl-dev libnewt-dev \
      libsrtp2-dev libspeex-dev libspeexdsp-dev \
      libgmime-3.0-dev libunbound-dev libusb-1.0-0-dev \
      libogg-dev libvorbis-dev libasound2-dev \
      libcurl4-openssl-dev liburiparser-dev \
      liblua5.2-dev libxslt1-dev libsystemd-dev \
      pkg-config xmlstarlet ca-certificates \
      autoconf automake libtool sudo bzip2 && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src

# ----------------------------
# Build and install pjproject (for PJSIP)
# ----------------------------
RUN git clone https://github.com/pjsip/pjproject.git && \
    cd pjproject && \
    git checkout 2.13 && \
    ./configure --prefix=/usr --libdir=/usr/lib/x86_64-linux-gnu \
        --enable-shared --disable-sound --disable-video \
        --disable-opencore-amr && \
    make -j$(nproc) && make install && ldconfig

# ----------------------------
# Build and install Asterisk 18.22.0
# ----------------------------
RUN git clone -b 18 https://github.com/asterisk/asterisk.git asterisk && \
    cd asterisk && \
    git checkout tags/18.22.0 -b build && \
    ./configure --with-pjproject --with-ssl --with-jansson && \
    make menuselect.makeopts && \
    menuselect/menuselect --enable CORE-SOUNDS-EN-WAV \
                          --enable MOH-OPSOUND-WAV \
                          --enable EXTRA-SOUNDS-EN-WAV \
                          menuselect.makeopts && \
    make -j$(nproc) && make install && make samples && make config && ldconfig

# ----------------------------
# Networking
# ----------------------------
EXPOSE 5060/udp 5060/tcp 8088/tcp

# Working directory for runtime
WORKDIR /etc/asterisk

# Run Asterisk in foreground as root
ENTRYPOINT ["/usr/sbin/asterisk", "-f", "-U", "root"]
