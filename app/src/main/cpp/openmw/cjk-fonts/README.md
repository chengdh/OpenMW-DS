# CJK font and encoding assets

## Contents

- `fonts/LXGWWenKaiGB-Regular.ttf` - LXGW WenKai GB, SIL Open Font License 1.1.
  Full text in `OFL.txt`. Source: https://github.com/lxgw/LxgwWenKaiGB
- `fonts/zh_CN.omwfont` - MyGUI TrueType font definition with code range
  33-65535, covering the CJK ideographs used by the tes3cn localisation.

## Build integration

`app/src/main/cpp/openmw/CMakeLists.txt` copies `fonts/` into
`resources/vfs/fonts` and applies `patches/openmw/cjk-encoding-support.patch`
to the OpenMW source tree (generated against the pinned `OPENMW_VERSION`).
`app/openmw.cfg` enables the font with `fallback=Fonts_Font_0,zh_CN`; remove
that line to keep the original Morrowind bitmap font.

## Encoding modes

| `encoding=` in openmw.cfg | Converter | Use case |
| --- | --- | --- |
| `gbk` | ICU (`ucnv_convert`), libiconv fallback | Existing GBK/CP936 `tes3cn_*.esp` localisations |
| `utf8` | none (passthrough) | UTF-8 content files, e.g. the UTF-8 variants of `tes3cn_*.esp` from amazingfate/tes3-chinese-translation |

ICU is the primary converter because the Android build always links it
(`OPENMW_USE_SYSTEM_ICU=ON`, `find_package(ICU REQUIRED COMPONENTS uc i18n
data)`) while bionic ships no iconv at all. `libiconv` is used on desktop
targets that have it. On a target with neither, build with
`-DOPENMW_CJK_NO_ICONV`: GBK then passes bytes through unchanged instead of
failing to link, and the dependency-free `utf8` mode keeps working.

The launcher charset menu lists only the names the engine accepts
(`win1252`, `win1250`, `win1251`, `gbk`, `utf8`).

## Verification status

- `patch -p1 --dry-run` against the pinned OpenMW commit: clean.
- C++20 syntax check in all three converter modes (ICU / iconv / none): clean.
- ICU GBK -> UTF-8 conversion and the reverse round-trip: byte-exact.
- A CI build of the APK and on-device rendering still have to confirm the
  packaged result; see `.github/workflows/build-cjk-apk.yml`.
