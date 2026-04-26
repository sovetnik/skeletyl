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
  - `T` -> `&rgb_ug RGB_TOG`
  - `A/S/D/F` -> `&bt BT_SEL 0/1/2/3`
  - `G/B` -> `&rgb_ug RGB_ON` / `&rgb_ug RGB_OFF`
- Added conditional mirror layer `MIR` (`SYM + ALT`) with right-hand recovery mirror and RGB controls:
  - `P` -> `&sys_reset`
  - `O` -> `&bootloader`
  - `I` -> `&bt BT_CLR_ALL`
  - `U` -> `&out OUT_USB`
  - `Y` -> `&rgb_ug RGB_TOG`
  - `H/N` -> `&rgb_ug RGB_EFF` / `&rgb_ug RGB_EFR`
  - RGB matrix on left side:
    - `A/Z` saturation `+/-`
    - `S/X` hue `+/-`
    - `D/C` speed `+/-`
    - `F/V` brightness `+/-`
- Layer display names expanded for Studio:
  - `DEFAULT`, `GAME`, `GAME_FN`, `SYMBOLS`, `NAVIGATION`, `NUMPAD`, `ALT_CHARS`, `COMMAND`, `MIRROR`
- Thumb mod-taps on base layer aligned with ergonaut style:
  - left thumb: `&mt LSHFT SPACE`
  - right thumb: `&mt RSHFT DEL`
- Added brightness controls in `COMMAND` layer on right edge:
  - `P` -> `C_BRI_UP`
  - key below -> `C_BRI_DEC`
- Added Elixir macro cluster on `ALT_CHARS`:
  - `S` -> `m_elpipe` (`|>`)
  - `D` -> `m_ldsh` (`<-`)
  - `F` -> `m_rdsh` (`->`)
  - `X` -> `m_elmap` (`%{`)
  - `C` -> `m_leq` (`<=`)
  - `V` -> `m_req` (`=>`)
  - `B` -> `m_insl` (`insl`)
- Refactor: extracted behavior/macro definitions from keymap to separate file:
  - `boards/arm/skeletyl/skeletyl_behaviors.dtsi`
  - `skeletyl.keymap` now keeps imports, conditional layers, and layer bindings.
