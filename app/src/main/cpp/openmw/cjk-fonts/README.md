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

Verified end to end on an AYN Thor (Android 13, arm64-v8a) with the
`tes3cn_*.esp` GBK localisation plugins loaded:

- `patch -p1 --dry-run` against the pinned OpenMW commit: clean.
- C++20 syntax check in all three converter modes (ICU / iconv / none): clean.
- ICU GBK -> UTF-8 conversion and the reverse round-trip: byte-exact.
- `openmw.log` reports `Using Chinese (GBK) font encoding.` and
  `Loading TrueType font file fonts/zh_cn.omwfont`.
- In game the main menu (新游戏 / 选项 / 载入游戏 / 退出), the character
  naming dialog (名字 / 确定) and the tutorial message
  (按下 W 键和 S 键前后移动，按下 A 键和 D 键左右横移，拖动鼠标来移动你的视角。)
  all render as clean Chinese, with no mojibake and no missing-glyph boxes.

Two workflows produce the APK: `build-cjk-apk.yml` builds the patched engine
from source, `hybrid-apk.yml` swaps this engine into the current upstream
release as a fast smoke test.

## Packaging caveat: AGP can drop the CMake-generated assets

CMake copies the engine resources into `generated_assets/libopenmw`. When the
Gradle build directory is relocated to another mount (the CI workflow moves it
to `/mnt/work` to reclaim disk), `mergeDebugAssets` has already taken its
snapshot, so the finished APK contains no `assets/libopenmw` at all and the
engine aborts at startup with `FileNotFoundException: libopenmw/resources`.

Both workflows therefore inspect the packaged APK and, when the engine assets
are missing, add them back with `zip -X` before `zipalign -p 4` and
`apksigner`. Signing uses a cached debug keystore (`cjk-debug-keystore-v1`) so
consecutive builds keep the same signature and install as updates.

## Trying it

1. Install the GBK `tes3cn_*.esp` plugins together with the rest of your
   Morrowind `Data Files` and register them in `openmw.cfg` as usual.
2. Add `encoding=gbk` to `openmw.cfg` - the launcher's Settings > Encoding
   menu offers the same list.
3. Install the APK from one of the workflows above and launch it once; the
   bundled resources, including the font, are unpacked into
   `/storage/emulated/0/OpenMW-DS/resources`.

Use `encoding=utf8` instead when your localisation files are already UTF-8,
for example the UTF-8 variants of `tes3cn_*.esp`.
