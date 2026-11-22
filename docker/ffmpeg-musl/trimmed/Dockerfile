# syntax=docker/dockerfile:1
#
# Trimmed FFmpeg 9.1 for Alpine/musl. Only the codecs, filters, protocols and
# hardware paths the bot actually uses. The full variant remains the safe
# fallback in the same release. Both are built against Alpine edge libraries;
# this one uses --disable-autodetect so no optional library leaks in.

FROM alpine:edge AS ffmpeg-builder

ARG FFMPEG_VERSION=9.1
ARG FFMPEG_SOURCE_URL=""
ARG FFMPEG_SHA256=1a4d394ae63d190a526cd052eb4415c156b5c5e2df37efab41a7de0b8fde2433

RUN set -eux; \
    apk add --no-cache \
        build-base pkgconf linux-headers curl xz tar ca-certificates \
        x264-dev x265-dev libvpx-dev dav1d-dev libwebp-dev \
        lame-dev opus-dev libvorbis-dev \
        freetype-dev fontconfig-dev fribidi-dev harfbuzz-dev libass-dev \
        libxml2-dev openssl-dev zlib-dev libva-dev libdrm-dev libx11-dev; \
    case "$(uname -m)" in \
        x86_64) apk add --no-cache nasm libvpl-dev ;; \
    esac

RUN set -eux; \
    SOURCE_URL="${FFMPEG_SOURCE_URL:-https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.xz}"; \
    curl --fail --location --proto '=https' --tlsv1.2 --retry 3 --retry-delay 2 \
        -o /tmp/ffmpeg.tar "$SOURCE_URL"; \
    if [ -n "$FFMPEG_SHA256" ] && [ "$FFMPEG_SHA256" != "skip" ]; then \
        echo "${FFMPEG_SHA256}  /tmp/ffmpeg.tar" | sha256sum -c -; \
    fi; \
    mkdir -p /tmp/ffsrc; \
    tar -xf /tmp/ffmpeg.tar -C /tmp/ffsrc; \
    cd "$(find /tmp/ffsrc -mindepth 1 -maxdepth 1 -type d | head -1)"; \
    if [ "$FFMPEG_VERSION" = "master" ]; then printf 'master\n' > RELEASE; fi; \
    VPL_FLAG=""; \
    if [ "$(uname -m)" = "x86_64" ]; then VPL_FLAG="--enable-libvpl"; fi; \
    ./configure \
        --prefix=/opt/ffmpeg \
        --disable-autodetect \
        --disable-debug \
        --disable-doc \
        --disable-ffplay \
        --disable-librtmp \
        --disable-lzma \
        --disable-static \
        --enable-shared \
        --enable-pic \
        --enable-pthreads \
        --enable-gpl \
        --enable-version3 \
        --enable-avfilter \
        --enable-openssl \
        --enable-zlib \
        --enable-libass \
        --enable-libdav1d \
        --enable-libdrm \
        --enable-libfontconfig \
        --enable-libfreetype \
        --enable-libfribidi \
        --enable-libharfbuzz \
        --enable-libmp3lame \
        --enable-libopus \
        --enable-libvorbis \
        --enable-libvpx \
        --enable-libwebp \
        --enable-libx264 \
        --enable-libx265 \
        --enable-libxml2 \
        --enable-vaapi \
        --enable-lto=auto \
        --optflags="-O3" \
        --extra-ldflags="-Wl,-rpath,/opt/ffmpeg/lib" \
        --extra-ldexeflags="-Wl,-rpath,/opt/ffmpeg/lib" \
        $VPL_FLAG; \
    make -j"$(nproc)"; \
    make install; \
    cp COPYING* /opt/ffmpeg/; \
    strip --strip-unneeded /opt/ffmpeg/bin/ffmpeg /opt/ffmpeg/bin/ffprobe; \
    find /opt/ffmpeg/lib -type f -name '*.so.*' -exec strip --strip-unneeded {} +; \
    rm -rf /opt/ffmpeg/include /opt/ffmpeg/share /opt/ffmpeg/lib/pkgconfig; \
    /opt/ffmpeg/bin/ffmpeg -version; \
    /opt/ffmpeg/bin/ffprobe -version; \
    rm -rf /tmp/ffmpeg* /tmp/ffsrc

FROM scratch AS artifact
COPY --from=ffmpeg-builder /opt/ffmpeg /opt/ffmpeg
