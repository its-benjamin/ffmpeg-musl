# ffmpeg-musl

FFmpeg builds for Alpine/musl, published as GitHub Release tarballs.
Built in GitHub Actions; the bot only downloads and verifies them.

## Versions

| Asset version | Source |
| --- | --- |
| `master` | FFmpeg master branch (floating tarball) |
| `9.1` | 9.1 |
| `9.0` | 9.0.2 |
| `8.1` | 8.1.3 |
| `8.0` | 8.0.3 |
| `7.1` | 7.1.5 |
| `7.0` | 7.0.3 |

Two variants per version:

- **full** — Alpine community/ffmpeg feature set: all codecs, filters, devices,
  and protocols Alpine ships.
- **trimmed** — only the codecs/filters the bot uses: x264/x265, vpx, dav1d,
  webp, opus/vorbis/mp3lame, fontconfig/freetype/fribidi/harfbuzz, libass,
  libxml2, OpenSSL, zlib, VAAPI, and libvpl on x86_64.

Assets:

```
ffmpeg-<version>-musl-<variant>-<arch>.tar.xz
SHA256SUMS
```

`<arch>` is `x86_64` or `aarch64`. Tarballs contain `ffmpeg/bin/ffmpeg`,
`ffmpeg/bin/ffprobe`, `ffmpeg/lib/libav*.so*`, and FFmpeg `COPYING.*`.
Binaries carry `-Wl,-rpath,/opt/ffmpeg/lib`; `ffplay`, docs, and debug symbols
are not included. x86_64 enables VAAPI and QSV (libvpl); aarch64 enables VAAPI.

Not built: 6.x, 5.x, 4.4 (they need older toolchains/dependency versions), and
LGPL/nonfree variants (the bot needs GPL x264/x265).

7.0/7.1 full builds omit the Vulkan flag (not present in those configure
scripts) and disable libv4l2 when Alpine's `posix-ioctl.patch` does not apply.
Everything else uses the same dependency/configure set as 8.x/9.x.

## Build

Automatic triggers:

- push touching `docker/ffmpeg-musl/**` or the workflow
- weekly schedule (`0 4 * * 1`)
- manual `workflow_dispatch`

28 jobs (7 versions × 2 variants × 2 architectures), limited to 20 concurrent
on standard public runners. Release source versions are SHA-256 pinned; master
is a floating branch tarball. Release tag: `musl-latest`.

## Install

```sh
BASE=https://github.com/its-benjamin/ffmpeg-musl/releases/download/musl-latest
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/ffmpeg-9.1-musl-trimmed-x86_64.tar.xz"
sha256sum -c --ignore-missing SHA256SUMS
mkdir -p /opt
tar -xJf ffmpeg-9.1-musl-trimmed-x86_64.tar.xz -C /opt
/opt/ffmpeg/bin/ffmpeg -version
```

The checksum line for the downloaded tarball must print `OK`. `musl-latest` is
rebuilt; keep the `SHA256SUMS` values for a fixed reference.

## Runtime libraries

Install the Alpine edge packages for the variant before running.

Trimmed:

```
libgcc x264-libs x265-libs libvpx dav1d libwebp libwebpmux lame-libs opus
libvorbis freetype fontconfig fribidi harfbuzz libass libxml2 libcrypto3
libssl3 zlib libva libdrm libx11
```

Add `libvpl` on x86_64. Full uses the `ffmpeg-libav*` dependency set:

```
libgcc libSvtAv1Enc aom-libs libdav1d libjxl lame-libs opus rav1e-libs
libtheora libvorbis libvpx libwebp libwebpmux x264-libs x265-libs xvidcore
zlib alsa-lib libdrm libpulse sndio-libs v4l-utils-libs libxcb libass
freetype fontconfig fribidi harfbuzz lilv-libs libplacebo vidstab zimg libzmq
libbluray libbz2 libcrypto3 libssl3 libdvdnav libdvdread libopenmpt librist
libsrt libssh libxml2 libx11 libvdpau soxr libva vulkan-loader
```

Add `libvpl`, `intel-media-driver`, and `onevpl-intel-gpu` on x86_64 for
QSV/VAAPI. These are Alpine/musl libraries; the binaries do not run on glibc.

The bot pins `musl-9.1` and defaults to the trimmed x86_64 asset.

## License

FFmpeg is built with `--enable-gpl`; license: GPL-3.0-or-later. `COPYING.*`
files are included in every tarball.
