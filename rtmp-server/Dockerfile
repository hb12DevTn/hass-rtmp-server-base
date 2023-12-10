ARG BUILD_FROM=ghcr.io/hassio-addons/base-python:12.0.2
# hadolint ignore=DL3006
FROM ${BUILD_FROM}


# Versions of nginx, rtmp-module and ffmpeg 
ARG  NGINX_VERSION=1.17.5
ARG  NGINX_RTMP_MODULE_VERSION=1.2.1
ARG  FFMPEG_VERSION=4.2.1

# Install dependencies
RUN apk update	&& \
	apk --no-cache add \
		bash build-base ca-certificates \
		openssl openssl-dev make \
		gcc libgcc libc-dev rtmpdump-dev \
		zlib-dev musl-dev pcre pcre-dev lame-dev \
		yasm pkgconf pkgconfig libtheora-dev \
		libvorbis-dev libvpx-dev freetype-dev  \
		x264-dev x265-dev && \
    rm -rf /var/lib/apt/lists/*	
	
# Download nginx source
RUN mkdir -p /tmp/build

WORKDIR /tmp/build

RUN	wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
	tar zxf nginx-${NGINX_VERSION}.tar.gz && \
	rm nginx-${NGINX_VERSION}.tar.gz

# Download rtmp-module source
RUN wget -q https://github.com/arut/nginx-rtmp-module/archive/v${NGINX_RTMP_MODULE_VERSION}.tar.gz && \
    tar zxf v${NGINX_RTMP_MODULE_VERSION}.tar.gz && \
	rm v${NGINX_RTMP_MODULE_VERSION}.tar.gz
	

# Build nginx with nginx-rtmp module
WORKDIR  /tmp/build/nginx-${NGINX_VERSION}

RUN ./configure \
        --sbin-path=/usr/local/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \		
        --pid-path=/var/run/nginx/nginx.pid \
        --lock-path=/var/lock/nginx.lock \
        --http-client-body-temp-path=/tmp/nginx-client-body \
        --with-http_ssl_module \
        --with-threads \
        --add-module=/tmp/build/nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION} && \
    make CFLAGS=-Wno-error -j $(getconf _NPROCESSORS_ONLN) && \
    make install

# Download ffmpeg source
WORKDIR /tmp/build

RUN wget -q https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.gz && \
  tar zxf ffmpeg-${FFMPEG_VERSION}.tar.gz && \
  rm ffmpeg-${FFMPEG_VERSION}.tar.gz

# Build ffmpeg
WORKDIR /tmp/build/ffmpeg-${FFMPEG_VERSION}

RUN ./configure \
	  --enable-version3 \
	  --enable-gpl \
	  --enable-small \
	  --enable-libx264 \
	  --enable-libx265 \
	  --enable-libvpx \
	  --enable-libtheora \
	  --enable-libvorbis \
	  --enable-librtmp \
	  --enable-postproc \
	  --enable-avresample \
	  --enable-swresample \ 
	  --enable-libfreetype \
	  --enable-libmp3lame \
	  --disable-debug \
	  --disable-doc \
	  --disable-ffplay \
	  --extra-libs="-lpthread -lm" && \
	make -j $(getconf _NPROCESSORS_ONLN) && \
	make install	

# Copy stats.xsl file to nginx html directory and clean build files
RUN cp /tmp/build/nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION}/stat.xsl /usr/local/nginx/html/stat.xsl

# After building, remove unnecessary packages and files
RUN rm -rf /var/cache/apk/* /tmp/build


RUN apk update	&& \
	apk --no-cache add \
		bash ca-certificates openssl \
		pcre libtheora libvorbis libvpx \
		librtmp x264-dev x265-dev freetype lame && \
    rm -rf /var/lib/apt/lists/*	

# Forward logs to Docker
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

# Copy  nginx config file to container
COPY nginx.conf /etc/nginx/nginx.conf


EXPOSE 1935
EXPOSE 8080

# CMD
CMD ["nginx", "-g", "daemon off;"]

# Build arugments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION
ARG BUILD_REPOSITORY

# Labels
LABEL \
	io.hass.name="Addon RTMP Server base for ${BUILD_ARCH}" \
    io.hass.description="Home Assistant Add-on: ${BUILD_ARCH} RTMP Server base image" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="base" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Hamdi Bougattaya <h.bougattaya1212@gmail.com>" \
    org.opencontainers.image.title="Addon RTMP Server for ${BUILD_ARCH}" \
    org.opencontainers.image.description="RTMP Server Home Assistant Add-on: ${BUILD_ARCH}" \
    org.opencontainers.image.vendor="Hamdi Bougattaya Add-ons" \
    org.opencontainers.image.authors="Hamdi Bougattaya <h.bougattaya1212@gmail.com>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/master/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}