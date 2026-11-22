# ffmpeg-musl

FFmpeg 9.1 builds for Alpine/musl, published as GitHub Release tarballs.
Built in GitHub Actions on standard public runners; no local build step.

## Assets

| File | Variant |
| --- | --- |
| `ffmpeg-9.1-musl-full-x86_64.tar.xz` | full, x86_64 |
| `ffmpeg-9.1-musl-full-aarch64.tar.xz` | full, aarch64 |
| `ffmpeg-9.1-musl-trimmed-x86_64.tar.xz` | trimmed, x86_64 |
| `ffmpeg-9.1-musl-trimmed-aarch64.tar.xz` | trimmed, aarch64 |
| `SHA256SUMS` | checksums for all four tarballs |

- **full** — Alpine community/ffmpeg feature set: all codecs, filters, devices
  and protocols Alpine ships.
- **trimmed** — only the codecs/filters the bot uses: x264/x265, vpx, dav1d,
  webp, opus/vorbis/mp3lame, fontconfig/freetype/fribidi/harfbuzz, libass,
  libxml2, OpenSSL, zlib, VAAPI, and libvpl on x86_64.

Both are FFmpeg 9.1 release sources, `--enable-gpl`, shared `libav*`
libraries, `-Wl,-rpath,/opt/ffmpeg/lib`. `ffplay`, docs and debug symbols are
not included. x86_64 builds enable VAAPI and QSV (libvpl); aarch64 builds
enable VAAPI.

The `musl-9.1` release tag is rebuilt on build-file changes and weekly. For a
fixed reference, keep the `SHA256SUMS` values you verify.

## Build

Automatic triggers:

- push touching `docker/ffmpeg-musl/**` or `.github/workflows/ffmpeg-musl.yml`
- weekly schedule (`0 4 * * 1`)
- manual `workflow_dispatch`

The matrix builds full/trimmed on `ubuntu-24.04` and `ubuntu-24.04-arm`
(native runners). FFmpeg version and source checksum are pinned in each
Dockerfile via `FFMPEG_VERSION` / `FFMPEG_SHA256`.

## Install

```sh
BASE=https://github.com/its-benjamin/ffmpeg-musl/releases/download/musl-9.1
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/ffmpeg-9.1-musl-trimmed-x86_64.tar.xz"
sha256sum -c --ignore-missing SHA256SUMS
mkdir -p /opt
tar -xJf ffmpeg-9.1-musl-trimmed-x86_64.tar.xz -C /opt
/opt/ffmpeg/bin/ffmpeg -version
```

The checksum line for the downloaded tarball must print `OK`.

## Runtime libraries

The tarballs contain only `ffmpeg/` (binaries, `libav*` shared libraries and
the FFmpeg `COPYING.*` files). Install the Alpine edge packages matching the
variant before running:

- **trimmed**: `libgcc x264-libs x265-libs libvpx dav1d libwebp libwebpmux
  lame-libs opus libvorbis freetype fontconfig fribidi harfbuzz libass
  libxml2 libcrypto3 libssl3 zlib libva libdrm libx11`; add `libvpl` on
  x86_64.
- **full**: the `ffmpeg-libav*` dependency set: `libgcc libSvtAv1Enc aom-libs
  libdav1d libjxl lame-libs opus rav1e-libs libtheora libvorbis libvpx
  libwebp libwebpmux x264-libs x265-libs xvidcore zlib alsa-lib libdrm
  libpulse sndio-libs v4l-utils-libs libxcb libass freetype fontconfig
  fribidi harfbuzz lilv-libs libplacebo vidstab zimg libzmq libbluray libbz2
  libcrypto3 libssl3 libdvdnav libdvdread libopenmpt librist libsrt libssh
  libxml2 libx11 libvdpau soxr libva vulkan-loader`; add `libvpl`,
  `intel-media-driver` and `onevpl-intel-gpu` on x86_64 for QSV/VAAPI.

These are Alpine/musl libraries. The binaries do not run on glibc systems.

## License

FFmpeg is built with `--enable-gpl`; license: GPL-3.0-or-later. The
`COPYING.*` files are included in each tarball.
