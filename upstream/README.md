# Folder structure

- `series/`: Individual iterations of the patch series (see changelog below). Each subfolder (e.g. `v0.1/`, `v0.2/`) contains the multi-file patch set for that version (including commit messages and cover letters).
- `combined/`: Single-file versions of each series iteration. These are combined patches applied to a specific kernel release (e.g., v6.19.14), to make it easy to test the series with `git apply` or `patch -p1`.

# Changelog

## v0.2.3

- Add another pincfg override in the realtek fixup: `{ 0x17, 0x90170111 }`. This is needed on the Pro 5 to enable the woofers, as the corresponding pin complex is wrongly reported as unconnected (`Pin Default 0x411111f0`, as reported by tester logs). This matches the pattern of `alc245_fixup_hp_spectre_x360_eu0xxx`, `alc245_fixup_hp_spectre_x360_eu0xxx`, and `alc287_fixup_yoga9_14iap7_bass_spk_pin`. Association `11` ensures no conflicts happen with speakerbar/tweeters node `0x14` (default association `10` on both pro 5 and 7) and headphones node `0x21` (`20` on pro 5, `1f` on pro 7).
- Add legion aw88399 entry in `alc269_fixup_models` to help debug new devices in the future:
```c
{.id = ALC287_FIXUP_LENOVO_LEGION_AW88399, .name = "alc287-lenovo-legion-aw88399"},
```
- Updated commit messages to match these changes, plus some slight rewording here and there.
- Matched `dev_info` string convention in property driver quirks.

## v0.2.2

- Added experimental support for Legion Pro 5 16IAX10H `0x17aa3908` (same realtek and property driver quirks as the Pro 7, `17AA3908` SSID in property driver).
- Changed name of `0x17aa3d6c` quirk to `"Legion Pro 7i 16IAX10H / Y9000P IAX10"`.
- Rebased on commit `b8dc547edf9e41474d8ce2dcf344e8e75b17781a` from `tiwai/sound`.

## v0.2.1
See [here](https://github.com/nadimkobeissi/16iax10h-linux-sound-saga/issues/55#issuecomment-4410686082) for more informations.

- In `alc269.c`, separated the `SND_PCI_QUIRK` entry for the Lenovo Legion Y9000P IAX10 (realtek PCI SSID `0x3d6c`) from the Legion Pro 7i 16IAX10H (`0x3907`). Tester logs confirmed that the Y9000P uses PCI SSID `0x3d6c` but ACPI SSID `17AA3907` for the AW88399, so the realtek quirk table needs a separate entry while the property table does not. This also fixes checkpatch line-length warning caused by the previous combined quirk table entry.
- Removed `17AA3D6C` from the AW88399 property table, as no machine has actually been observed to use it as an ACPI subsystem ID.
- Added `Tested-by: Xia Yun'an <imitoy@imitoy.top>` (Lenovo Legion Y9000P IAX10, kernel 7.0.3, Arch Linux).
- Consistent include guard style in `aw88399_hda_property.h` (fixed missing double underscores). Also backported to v0.2.
- Changed include guard in `include/sound/aw88399.h` to `__SOUND_AW88399_H`.
- In `aw88399-lib.c`, changed `MODULE_DESCRIPTION` from `AW88399 library` to `AW88399 common device library`.
- In `aw88399-lib.c`, changed `MODULE_LICENSE` from the original `"GPL v2"` to `"GPL"` to fix warning from `scripts/checkpatch.pl`.
- Rebased on commit `b8dc547edf9e41474d8ce2dcf344e8e75b17781a` from `tiwai/sound`.

## v0.2
See [here](https://github.com/nadimkobeissi/16iax10h-linux-sound-saga/issues/55#issuecomment-4381559698) for more informations.

- Reworked patch 1 to introduce a common header at `include/sound/aw88399.h`. This file includes all the definitions from the original `sound/soc/codecs/aw88399.h` not strictly related to ASoC. As a result, `sound/soc/codecs/aw88399-lib.h` has been removed, `sound/soc/codecs/aw88399.h` heavily depleted, and most of the ugly imports in `sound/hda/codecs/side-codecs/aw88399_hda.c` have been removed in favor of a single `#include <sound/aw88399.h>` in `sound/hda/codecs/side-codecs/aw88399_hda.h`.
- Added `Tested-by: Nadim Kobeissi <nadim@symbolic.software>`.
- Rebased the patch series on commit `fac9a31701803e4e41fdb7b5c71582c65cf47176` from `tiwai/sound`.

## v0.1
Original patch series proposal, based on commit `876c495d412ef67bd4d0bdc4b74b0bd3d9f4e890` from `tiwai/sound`.

See [here](https://github.com/nadimkobeissi/16iax10h-linux-sound-saga/issues/55#issue-4343077194) for more informations.

1. **ASoC: aw88399: extract shared device library** - pure code movement, no logic changes
2. **ASoC: aw88399: check return values in aw_dev_check_sram**: - proper error handling for DSP access calls (bugfix)
3. **ASoC: aw88399: derive channel from I2C address on ACPI systems**
4. **ASoC: aw88399: add per-instance BSTS status bypass flag**
5. **ACPI/platform: add AWDZ8399 to serial-multi-instantiate**
6. **ALSA: hda/scodec: add AW88399 HDA side codec driver** - the new driver
7. **ALSA: hda/realtek: enable AW88399 on Lenovo Legion Pro 7** - property driver and realtek quirks
