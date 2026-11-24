# ffmpeg-musl

FFmpeg builds for Alpine/musl. **master is autobuilt daily**; 9.1 weekly.
The bot uses the master trimmed asset.

## Versions

| Asset version | Source | Release | Cadence |
| --- | --- | --- | --- |
| `master` | FFmpeg master (floating branch tarball) | `autobuild-master-*` (Latest) | daily |
| `9.1` | 9.1 release tarball (SHA-256 pinned) | `autobuild-9.1-*` | weekly |

Older 7.x/8.x/9.0 builds were removed to keep CI bounded and avoid
toolchain/API drift.

Two variants per version:

- **full** — Alpine community/ffmpeg feature set (all codecs, filters, devices,
  and protocols Alpine ships).
- **trimmed** — only the codecs/filters the bot uses: x264/x265, vpx, dav1d,
  webp, opus/vorbis/mp3lame, fontconfig/freetype/fribidi/harfbuzz, libass,
  libxml2, OpenSSL, zlib, VAAPI, and libvpl on x86_64.

Assets:

```
ffmpeg-master-musl-<variant>-<arch>.tar.xz   (dated master releases)
ffmpeg-9.1-musl-<variant>-<arch>.tar.xz      (dated 9.1 releases)
SHA256SUMS
```

`<arch>` is `x86_64` or `aarch64`. Tarballs contain `ffmpeg/bin/ffmpeg`,
`ffmpeg/bin/ffprobe`, `ffmpeg/lib/libav*.so*`, and FFmpeg `COPYING.*`.
Binaries carry `-Wl,-rpath,/opt/ffmpeg/lib`; `ffplay`, docs, and debug symbols
are not included. x86_64 enables VAAPI and QSV (libvpl); aarch64 enables VAAPI.

## Releases

Every successful run publishes exactly one new dated release:

- `autobuild-master-YYYY-MM-DD-HH-MM` — daily master build, marked Latest
- `autobuild-9.1-YYYY-MM-DD-HH-MM` — weekly 9.1 build

Each release contains the four tarballs and `SHA256SUMS`. Only master
releases are marked Latest, so `releases/latest/download` always resolves the
newest master build. Pin a dated tag for reproducible installs.

Retention: the 14 newest dated releases and the first release of each month
for up to 24 months are kept. Older releases and tags are deleted.

## Build

- master: `.github/workflows/ffmpeg-master.yml`; daily `0 4 * * *`,
  Dockerfile changes, manual dispatch; 4 jobs; one dated
  `autobuild-master-*` release per run, marked Latest.
- 9.1: `.github/workflows/ffmpeg-musl.yml`; weekly `0 4 * * 1`, Dockerfile
  changes, manual dispatch; 4 jobs; one dated `autobuild-9.1-*` release per
  run, not marked Latest.

Standard public runners, native x86_64/aarch64. The master tarball floats;
release sources are SHA-256 pinned.

## Install

Master, newest build, trimmed:

```sh
BASE=https://github.com/its-benjamin/ffmpeg-musl/releases/latest/download
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/ffmpeg-master-musl-trimmed-x86_64.tar.xz"
sha256sum -c --ignore-missing SHA256SUMS
mkdir -p /opt
tar -xJf ffmpeg-master-musl-trimmed-x86_64.tar.xz -C /opt
/opt/ffmpeg/bin/ffmpeg -version
```

Pin a dated master build by replacing `latest/download` with
`download/autobuild-master-YYYY-MM-DD-HH-MM`.

9.1 is always pinned to a dated tag:

```sh
TAG=autobuild-9.1-YYYY-MM-DD-HH-MM
BASE=https://github.com/its-benjamin/ffmpeg-musl/releases/download/$TAG
curl -fsSLO "$BASE/SHA256SUMS"
curl -fsSLO "$BASE/ffmpeg-9.1-musl-trimmed-x86_64.tar.xz"
sha256sum -c --ignore-missing SHA256SUMS
```

The downloaded tarball must print `OK`.

## Runtime libraries

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
QSV/VAAPI. These are Alpine/musl libraries; binaries do not run on glibc.

## License

FFmpeg is built with `--enable-gpl`; license: GPL-3.0-or-later. `COPYING.*`
files are included in every tarball.
