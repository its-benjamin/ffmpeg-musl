# ffmpeg-musl

Prebuilt FFmpeg 9.1 runtime tarballs for Alpine/musl, published as GitHub
Release assets. The release layout follows
[BtbN/FFmpeg-Builds](https://github.com/FFmpeg-Builds/FFmpeg-Builds); the
multi-arch CI follows [wader/static-ffmpeg](https://github.com/wader/static-ffmpeg).
Unlike BtbN, these builds link Alpine's shared musl libraries, so VAAPI/QSV use
the system Intel drivers and the artifact stays small.

## Variants

| Variant | Contents |
| --- | --- |
| `full` | Alpine community/ffmpeg feature set: every codec, filter, device and protocol Alpine ships. Safe fallback. |
| `trimmed` | Only the codecs/filters the bot uses: x264/x265, vpx, dav1d, webp, opus/vorbis/mp3lame, fontconfig/freetype/fribidi/harfbuzz, ass, xml2, OpenSSL, zlib, VAAPI, libvpl (x86_64). |

Both are built from the official FFmpeg 9.1 release, with `--enable-gpl`,
shared libav* libraries and `/opt/ffmpeg` rpath. `ffplay`, docs and debug
symbols are omitted.

## Build

Actions → **Build FFmpeg 9.1 musl** → Run workflow. The matrix builds
`x86_64` and `aarch64` on native GitHub-hosted runners and publishes:

```
ffmpeg-9.1-musl-full-x86_64.tar.xz
ffmpeg-9.1-musl-full-aarch64.tar.xz
ffmpeg-9.1-musl-trimmed-x86_64.tar.xz
ffmpeg-9.1-musl-trimmed-aarch64.tar.xz
SHA256SUMS
```

No automatic builds: a release is built only on manual dispatch, so forks and
application deploys do not consume CI time.

## Use

```sh
curl -fsSLO https://github.com/its-benjamin/ffmpeg-musl/releases/download/musl-9.1-r1/SHA256SUMS
curl -fsSLO https://github.com/its-benjamin/ffmpeg-musl/releases/download/musl-9.1-r1/ffmpeg-9.1-musl-trimmed-x86_64.tar.xz
sha256sum -c --ignore-missing SHA256SUMS
mkdir -p /opt
tar -xJf ffmpeg-9.1-musl-trimmed-x86_64.tar.xz -C /opt
/opt/ffmpeg/bin/ffmpeg -version
```

Install the matching Alpine runtime library packages for the chosen variant
(the full list is the `ffmpeg-libav*` dependency closure; the trimmed list is
just the libraries linked by the trimmed build). The FFmpeg binaries carry
`-Wl,-rpath,/opt/ffmpeg/lib`, so no `LD_LIBRARY_PATH` is needed.

## License

FFmpeg is built with `--enable-gpl` (`GPL-3.0-or-later`). The `COPYING.*` files
are included in every tarball and in `/opt/ffmpeg`.
