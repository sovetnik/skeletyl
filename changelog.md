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
- Elixir macros adapted for `Universal Layout Ortho` symbol mapping:
  - switched from generic HID symbol keycodes to layout-specific combos (`RA(O)`, `RA(P)`, `LS(RA(T))`, `LS(LBKT)`)
  - fixed outputs for: `|>`, `<-`, `->`, `%{`, `<=`, `=>`.
- Added keymap-drawer pipeline (same pattern as ergonaut):
  - new workflow: `.github/workflows/draw-keymaps.yml`
  - source keymap: `boards/arm/skeletyl/skeletyl.keymap`
  - output artifacts: `keymap-drawer/skeletyl.yaml` and `keymap-drawer/skeletyl.svg`
  - forced drawing template: `foostan_corne_5col_layout` (3x5) to match skeletyl geometry.

## battery revive experiment

- Disabled battery reporting path while keeping split BLE experimental connection:
  - removed `CONFIG_ZMK_BATTERY_REPORTING=y` on both halves
  - removed `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y` on central
  - removed `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y` on central
- NAV layer adjustment:
  - `R` set to `&tabber`
  - `Q/W` set to `HOME/END`

## home-row mods (branch `homerow`, ported from charybdis)

Ported the home-row-mod pivot from the `charybdis` repo instead of keeping two
different modifier systems across the two personal keyboards. The
`cm_gui/cm_alt/cm_ctrl/cm_sft` sticky-mod chain (`sticky_forever` +
`m_press_key_twice`/`m_release_key_twice`) never crashed here - crashing was
specific to charybdis's dongle topology - but it's structurally the same
construction, so it's gone in favor of the simpler, already-verified-stable
recipe.

- `skeletyl_behaviors.dtsi`: removed `cm_gui/cm_alt/cm_ctrl/cm_sft`,
  `sticky_forever`, `m_press_key_twice`/`m_release_key_twice`, the
  `STICKY_FOREVER`/`MAX_TIMER` macros, and the global `&sk { release-after-ms
  = <60000>; }` override (all now dead). Added `hml`/`hmr` (timeless
  home-row mods: `balanced` flavor, `require-prior-idle-ms = <150>`,
  `tapping-term-ms = <280>`, `hold-trigger-on-release`).
  `THUMBS = 30 31 32 33 34 35` (skeletyl's thumb cluster is 3+3, unlike
  charybdis's 3+2) - both hands' three thumb keys included in both
  `hold-trigger-key-positions` lists, same as charybdis.
- `skeletyl.keymap` DEFAULT row1: `A S D F` / `J K L ;` now
  `&hml LGUI/LALT/LCTRL/LSHFT` / `&hmr LSHFT/LCTRL/LALT/LGUI`.
- NAV row1 left half: was `&cm_gui &cm_alt &cm_ctrl &cm_sft`, now mirrors the
  right half's arrows (`LEFT DOWN UP RIGHT RET` on both sides), matching
  charybdis.
- SYM row1 right half (`J K L ;`) and NUM row1 both halves (`A S D F` /
  `J K L ;`): former `&cm_*` positions now `______` (freed, not yet assigned
  - same open question as charybdis's equivalent slots).
- `la`'s `&kp LSHFT/LCTRL/LALT/LGUI` release-cleanup lines kept as-is
  (unchanged) - still load-bearing for `swapper_mac`/`tabber`, per the
  charybdis investigation.
