# CJK font and encoding assets

## Contents

- `fonts/LXGWWenKaiGB-Regular.ttf` - LXGW WenKai GB, licensed under the SIL
  Open Font License 1.1. Full text in `OFL.txt`. Source:
  https://github.com/lxgw/LxgwWenKaiGB
- `fonts/zh_CN.omwfont` - MyGUI TrueType font definition with code range
  33-65535, which covers the CJK ideographs used by the tes3cn localisation.

## Build integration

`app/src/main/cpp/openmw/CMakeLists.txt` copies `fonts/` into
`resources/vfs/fonts` and applies `patches/openmw/cjk-encoding-support.patch`
to the OpenMW source tree (the patch is generated against the pinned
`OPENMW_VERSION`). `app/openmw.cfg` enables the font with
`fallback=Fonts_Font_0,zh_CN`; remove that line to keep the original bitmap
font.

## Encoding modes

| `encoding=` in openmw.cfg | Requirement | Use case |
| --- | --- | --- |
| `gbk` | libiconv at link time | Existing GBK/CP936 `tes3cn_*.esp` localisations |
| `utf8` | none (passthrough) | UTF-8 content files, e.g. the UTF-8 variants of `tes3cn_*.esp` from amazingfate/tes3-chinese-translation |

If the Android target has no libiconv, build with `-DOPENMW_CJK_NO_ICONV`:
GBK then falls back to raw bytes without a link error, while the dependency-
free `utf8` mode keeps working. The launcher charset menu lists only the five
names the engine accepts (`win1252`, `win1250`, `win1251`, `gbk`, `utf8`).

## Verification status

The patch was checked with `patch -p1 --dry-run` against the pinned OpenMW
commit and C++20 syntax-checked both with and without libiconv. It has not yet
been compiled into an APK: a CI build is needed to confirm libiconv
availability and the runtime rendering of the new font.
