# Folder structure

- `series/`: Individual iterations of the patch series (see changelog below). Each subfolder (e.g. `v0.1/`, `v0.2/`) contains the multi-file patch set for that version (including commit messages and cover letters).
- `combined/`: Single-file versions of each series iteration. These are combined patches applied to a specific kernel release (e.g., v6.19.14), to make it easy to test the series with `git apply` or `patch -p1`.

# Changelog

## v0.2
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
