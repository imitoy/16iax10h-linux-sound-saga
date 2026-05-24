# Folder structure

- `series/`: Individual iterations of the patch series (see changelog below). Each subfolder (e.g. `v0.1/`, `v0.2/`) contains the multi-file patch set for that version (including commit messages and cover letters).
- `combined/`: Single-file versions of each series iteration. These are combined patches applied to a specific kernel release (e.g., v6.19.14), to make it easy to test the series with `git apply` or `patch -p1`.

# Changelog

## v0.3.1

- Added experimental support for Legion R9000P ADR10 (asian variant of the Pro 5 AMD that includes the aw88399 smart amp driving separate woofers, unlike the western variant which has only 2 speakers). The same realtek/property driver quirks are used for the other Legions currently supported.
The PCI SSID of this device is `0x17aa:38bb`, which is already present in `alc269.c` (`"Yoga S780-14.5 Air AMD quad AAC"`); therefore, the codec SSID (`0x17aa:3927/3928`) is used, and put before `38bb` (an exception to the usual sorting of this table) to prevent matching the wrong quirk before the right one is reached by the loop in `snd_hda_pick_fixup`.
- Reworked quirk matching for all other devices. Before, `SND_PCI_QUIRK` was used for every legion.
For the AMD models, this was slightly improper, because the codec SSID were passed, but it still worked in practice because there were no matches to the PCI SSID of these devices in `alc269.c` (`17aa:38c6`), so the `q = hda_quirk_lookup_id(codec_vendor, codec_device, quirk);` fallback in `snd_hda_pick_fixup` faced no problems.
For the Intel models, since both the codec SSIDs (`3906,3907`) and the PCI SSID (`3d6c`) were included, the for loop in `snd_hda_pick_fixup` was only matching the PCI SSID, so that the codec SSIDs were basically dead code.
Clean up this confusion by only matching the HDA codec IDs (consistently with the R9000P above) and removing the Intel model's PCI SSID.
- Rebased on commit `ef807cc07dec16edc7863d437e9250e20cb73741` from `tiwai/sound`.

## v0.3

See [here](https://github.com/nadimkobeissi/16iax10h-linux-sound-saga/issues/55#issuecomment-4508214865) for more informations.

- Added a new patch to the series (5/8) to introduce a firmware reload flag (`fw_needs_reload`) for system resume. After system sleep, the smart amp's SRAM loses its DSP firmware configuration (previously loaded at initialization).
In v<0.3 series, this would cause the first playback after system resume to fail a CRC check; the retry mechanism in `aw88399_start_pa` would then be triggered, re-uploading the firmware on the second attempt and finally succeeding. This sequence produces misleading error-level log messages, and while in practice capable of consistently self-correcting this failure, is technically an improper hijacking of a self-correct mechanism designed for a different purpose.
The new flag is set by the HDA driver during system suspend and checked by `aw88399_start` in the shared library, causing a proactive full firmware upload on the next playback start. This eliminates the spurious CRC failures and retry cycles after resume without affecting ASoC behavior.
- Slightly reworded the commit messages of patches 1 and 3 to make them more accurate.
- Rebased on commit `9b14f636834630e5473ee5020c8289823a481a7` from `tiwai/sound`.

## v0.2.4

- Reverted the addition of the Pro 5i ID 3908, as that should likely be sent as a separate patch.
- Added another entry in `alc269_fixup_models` to allow for testing of the aw88399 side codec driver decoupled from the realtek quirks:
```c
{.id = ALC287_FIXUP_AW88399_I2C_2, .name = "aw88399-i2c-2"},
```
- Added `Tested-by: Munzir Taha <munzirtaha@gmail.com>`.
- Rebased on commit `c19fcfd33d37d7979781ebe6cacc3c3da9ea0f2e` from `tiwai/sound`.

## v0.2.3

- Added another pincfg override in the realtek fixup: `{ 0x17, 0x90170111 }`. This is needed on the Pro 5 to enable the woofers, as the corresponding pin complex is wrongly reported as unconnected (`Pin Default 0x411111f0`, as reported by tester logs). This matches the pattern of `alc245_fixup_hp_spectre_x360_eu0xxx`, `alc245_fixup_hp_spectre_x360_eu0xxx`, and `alc287_fixup_yoga9_14iap7_bass_spk_pin`. Association `11` ensures no conflicts happen with speakerbar/tweeters node `0x14` (default association `10` on both pro 5 and 7) and headphones node `0x21` (`20` on pro 5, `1f` on pro 7).
- Added legion aw88399 entry in `alc269_fixup_models` to help debug new devices in the future:
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
