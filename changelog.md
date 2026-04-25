# changelog

## legacy exterminatus

Source: commit `25083f5` (`Update zmk to latest (#1)`).

- Update to zmk-latest.
- prepare to zephyr 4.1.
- fix: remove BOARD_* assignments from defconfigs.
- fix: drop SOC_* from board defconfigs.
- fix: clean kconfig warnings for zephyr 4.1.
- fix: remove deprecated bt controller tx power config.
- fix: set gpiote-instance for gpio ports.
- fix: enable gpiote for gpio nrfx.
- fix: disable usb for settings_reset build.
- fix: build settings_reset on right board.
- feat: set nrf low power clock config for sleep wake.
- chore: set idle 15m and sleep timeout 24h.

## keymap

Source: current staged changes in `boards/arm/skeletyl/skeletyl.keymap`.

- NAV layer: arrows moved to `HJKL` positions.
- Space switching macros aligned with macOS shortcuts:
  - `PSPC` -> `Ctrl+Left`
  - `NSPC` -> `Ctrl+Right`
- CMD layer (left side) repurposed for recovery/connectivity controls:
  - `Q` -> `&sys_reset`
  - `W` -> `&bootloader`
  - `E` -> `&bt BT_CLR_ALL`
  - `R` -> `&out OUT_USB`
  - `T` -> `&rgb_ug_tog`
  - `A/S/D/F` -> `&bt BT_SEL 0/1/2/3`
  - `G/B` -> `&rgb_ug_on` / `&rgb_ug_off`
- Added conditional mirror layer `MIR` (`SYM + ALT`) with right-hand recovery mirror and RGB controls:
  - `P` -> `&sys_reset`
  - `O` -> `&bootloader`
  - `I` -> `&bt BT_CLR_ALL`
  - `U` -> `&out OUT_USB`
  - `Y` -> `&rgb_ug_tog`
  - `H/N` -> `&rgb_ug_on` / `&rgb_ug_off`
  - RGB matrix on left side:
    - `A/Z` saturation `+/-`
    - `S/X` hue `+/-`
    - `D/C` speed `+/-`
    - `F/V` brightness `+/-`
