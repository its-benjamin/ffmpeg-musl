# syntax=docker/dockerfile:1
#
# Prebuilt FFmpeg 9.1 for Alpine/musl. Published by
# .github/workflows/ffmpeg-musl.yml and consumed by the bot image.
#
# This mirrors Alpine's community/ffmpeg feature set (all codecs, filters,
# devices and protocols) instead of trimming it, so replacing the distro
# 9.0.2 package cannot silently break a download/convert/music path. The
# only removals are ffplay, docs and debug symbols.
#
# BtbN-style release asset: a tar of the runtime tree (`bin/` + `lib/`).
# Unlike BtbN it links Alpine's shared musl libraries, so the artifact is
# small and hardware acceleration (VAAPI/QSV) uses the system drivers.

FROM alpine:edge AS ffmpeg-builder

ARG FFMPEG_VERSION=9.1
ARG FFMPEG_SOURCE_URL=""
ARG FFMPEG_SHA256=1a4d394ae63d190a526cd052eb4415c156b5c5e2df37efab41a7de0b8fde2433

RUN set -eux; \
    apk add --no-cache \
        build-base pkgconf linux-headers curl xz tar patch ca-certificates coreutils \
        alsa-lib-dev aom-dev bzip2-dev dav1d-dev fontconfig-dev freetype-dev \
        fribidi-dev harfbuzz-dev ladspa-dev lame-dev libass-dev libbluray-dev \
        libdrm-dev libdvdnav-dev libdvdread-dev libjxl-dev libopenmpt-dev \
        libplacebo-dev librist-dev libsrt-dev libssh-dev libtheora-dev libva-dev \
        libvdpau-dev libvorbis-dev libvpx-dev libwebp-dev libx11-dev libxcb-dev libxfixes-dev \
        libxml2-dev lilv-dev openssl-dev opus-dev perl-dev pulseaudio-dev \
        rav1e-dev sndio-dev soxr-dev v4l-utils-dev vidstab-dev \
        vulkan-loader-dev x264-dev x265-dev xvidcore-dev zeromq-dev zimg-dev zlib-dev; \
    case "$(uname -m)" in \
        x86_64) apk add --no-cache nasm libvpl-dev svt-av1-dev ;; \
        aarch64) apk add --no-cache svt-av1-dev ;; \
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
    curl --fail --location --proto '=https' --tlsv1.2 --retry 3 --retry-delay 2 \
        -o /tmp/posix-ioctl.patch \
        "https://raw.githubusercontent.com/alpinelinux/aports/master/community/ffmpeg/posix-ioctl.patch"; \
    V4L2_FLAG="--enable-libv4l2"; \
    if patch -p1 --dry-run < /tmp/posix-ioctl.patch >/dev/null 2>&1; then \
        patch -p1 < /tmp/posix-ioctl.patch; \
    elif patch -p1 -R --dry-run < /tmp/posix-ioctl.patch >/dev/null 2>&1; then \
        echo "posix-ioctl.patch already applied upstream"; \
    else \
        echo "posix-ioctl.patch does not apply; disabling libv4l2" >&2; \
        V4L2_FLAG="--disable-libv4l2"; \
    fi; \
    VULKAN_FLAG="--enable-vulkan"; \
    case "$FFMPEG_VERSION" in 7.*) VULKAN_FLAG="";; esac; \
    SVT_FLAG=""; \
    VPL_FLAG=""; \
    case "$(uname -m)" in \
        x86_64) SVT_FLAG="--enable-libsvtav1"; VPL_FLAG="--enable-libvpl" ;; \
        aarch64) SVT_FLAG="--enable-libsvtav1" ;; \
    esac; \
    ./configure \
        --prefix=/opt/ffmpeg \
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
        --enable-bzlib \
        --enable-iconv \
        --enable-ladspa \
        --enable-libaom \
        --enable-libass \
        --enable-libbluray \
        --enable-libdav1d \
        --enable-libdrm \
        --enable-libdvdnav \
        --enable-libdvdread \
        --enable-libfontconfig \
        --enable-libfreetype \
        --enable-libfribidi \
        --enable-libharfbuzz \
        --enable-libjxl \
        --enable-libmp3lame \
        --enable-libopenmpt \
        --enable-libopus \
        --enable-libplacebo \
        --enable-libpulse \
        --enable-librav1e \
        --enable-librist \
        --enable-libsoxr \
        --enable-libsrt \
        --enable-libssh \
        --enable-libtheora \
        --enable-libvidstab \
        --enable-libvorbis \
        --enable-libvpx \
        --enable-libwebp \
        --enable-libx264 \
        --enable-libx265 \
        --enable-libxcb \
        --enable-libxml2 \
        --enable-libxvid \
        --enable-libzimg \
        --enable-libzmq \
        --enable-lto=auto \
        --enable-lv2 \
        --enable-openssl \
        --enable-sndio \
        --enable-vaapi \
        --enable-vdpau \
        --enable-zlib \
        --optflags="-O3" \
        --extra-ldflags="-Wl,-rpath,/opt/ffmpeg/lib" \
        --extra-ldexeflags="-Wl,-rpath,/opt/ffmpeg/lib" \
        $SVT_FLAG $VPL_FLAG $V4L2_FLAG $VULKAN_FLAG; \
    make -j"$(nproc)"; \
    make install; \
    cp COPYING* /opt/ffmpeg/; \
    strip --strip-unneeded /opt/ffmpeg/bin/ffmpeg /opt/ffmpeg/bin/ffprobe; \
    find /opt/ffmpeg/lib -type f -name '*.so.*' -exec strip --strip-unneeded {} +; \
    rm -rf /opt/ffmpeg/include /opt/ffmpeg/share /opt/ffmpeg/lib/pkgconfig; \
    /opt/ffmpeg/bin/ffmpeg -version; \
    /opt/ffmpeg/bin/ffprobe -version; \
    rm -rf /tmp/ffmpeg* /tmp/ffsrc

# BtbN-style export: runtime tree only.
FROM scratch AS artifact
COPY --from=ffmpeg-builder /opt/ffmpeg /opt/ffmpeg
