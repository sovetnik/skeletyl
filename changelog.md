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

## nice!nano dongle + battery proxy (branch `homerow`, see rfcs/0001-nice-nano-dongle.md)

Brought skeletyl's topology in line with charybdis: a dongle as BLE central
instead of `skeletyl_left`, with full battery proxy for both halves. Unlike
charybdis (xiao_ble), the dongle here is an **Adafruit nice!nano** - what's
physically on hand for this build.

- Moved `boards/arm/skeletyl/skeletyl.keymap` and `skeletyl_behaviors.dtsi`
  to `config/` - ZMK's keymap search strips shield/board suffixes
  (`skeletyl_dongle`/`skeletyl_left`/`skeletyl_right` all reduce to
  `skeletyl`) and always checks `config/` first, so one file now serves all
  three build targets, same as `config/charybdis.keymap`.
- New shield `boards/shields/skeletyl_dongle/` targeting `nice_nano`:
  `Kconfig.shield`, `Kconfig.defconfig` (`ZMK_SPLIT_ROLE_CENTRAL=y`),
  `.conf` (2 peripherals, `CENTRAL_BATTERY_LEVEL_FETCHING`/`_PROXY`,
  `ZMK_SLEEP=n`), `.overlay`, `.zmk.yml`.
- The overlay's physical-layout node (`skeletyl_5col_layout` +
  `skeletyl_position_map`) is a hand-kept duplicate of the one in
  `boards/arm/skeletyl/skeletyl.dtsi`, not a shared include - that file also
  carries real board-level config (ADC, UART, flash partitions) that has no
  business being shield-visible, and cross-directory quote-includes don't
  resolve here (same class of problem fixed once already on charybdis).
  `zmk,kscan` on the dongle points at a `zmk,kscan-mock` node, matching
  `charybdis_dongle.overlay`.
- `boards/arm/skeletyl/Kconfig.defconfig`: removed
  `ZMK_SPLIT_BLE_ROLE_CENTRAL default y` from `BOARD_SKELETYL_LEFT` - both
  halves are peripherals now. `ZMK_USB default y` stays, so `skeletyl_left`
  can still be flashed standalone if ever needed.
- No split-queue-size bumps on the dongle (unlike charybdis) - skeletyl has
  no pointing device, so there's no continuous trackball traffic to size
  for; ZMK's default queue sizes are fine for ordinary keypresses.
- `build.yaml`: central build is now `nice_nano/nrf52840/zmk` +
  `skeletyl_dongle` (with Studio, same as before on `skeletyl_left`); halves
  build as before but no longer carry the Studio cmake-args.
- Added `skeletyl_right_standalone_central` fallback build (`skeletyl_right`
  with `-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y`), matching charybdis's
  `charybdis_right_standalone_central` - direct half-to-half BLE if the
  dongle is ever unavailable. Not yet confirmed on hardware.
- Fixed `.github/workflows/draw-keymaps.yml`: watched paths and the
  `keymap parse` source path both pointed at the old
  `boards/arm/skeletyl/skeletyl.keymap` location - updated to `config/`.

### CI fix: `skeletyl_left` link failure after the peripheral flip

`skeletyl_left`'s build failed at link time with undefined references to
`zmk_hid_get_keyboard_report`/`zmk_hid_get_consumer_report`/
`usb_hid_register_device`/`usb_hid_init` - all from `src/usb_hid.c`.

Root cause: `skeletyl_left_defconfig` had a literal `CONFIG_ZMK_USB=y`.
Upstream, `ZMK_USB` `depends on (!ZMK_SPLIT || ZMK_SPLIT_ROLE_CENTRAL)` and
`src/hid.c` (which provides those symbols) is only compiled under the same
condition (`app/CMakeLists.txt`) - but `target_sources_ifdef(CONFIG_ZMK_USB
... src/usb_hid.c)` still picked up the board-level forced value, compiling
`usb_hid.c` (which calls into `hid.c`) against a peripheral build that never
compiles `hid.c`. charybdis's peripheral `.conf` files never set `ZMK_USB`
at all and rely on `nice_nano`'s own board defconfig, which resolves
correctly for peripherals - removed the forced `CONFIG_ZMK_USB=y` from
`skeletyl_left_defconfig` and the now-pointless `config ZMK_USB / default y`
in `Kconfig.defconfig` to match.

### RGB underglow stopped responding to keymap hotkeys

Flashed and noticed the RGB glow hotkeys (MIR layer, CMD layer `RGB_TOG`)
did nothing on either half - previously (skeletyl_left as central) they
drove *both* halves' LEDs in sync, not just the triggering side.

Root cause, traced through ZMK's own behavior-dispatch code
(`src/behavior.c`): `&rgb_ug` is declared `BEHAVIOR_LOCALITY_GLOBAL`
(`behavior_rgb_underglow.c:260`) - invoking a global-locality behavior makes
the *central* fan it out to every split peripheral over BLE
(`zmk_split_central_invoke_behavior`, GATT `RUN_BEHAVIOR` characteristic in
`split/bluetooth/service.c`), which is what made both halves light up
together. But before any of that fan-out happens, the central has to
resolve `&rgb_ug` as a local behavior device first
(`zmk_behavior_get_binding` at the top of `zmk_behavior_invoke_binding`) -
and the dongle never had `CONFIG_ZMK_RGB_UNDERGLOW` enabled, so that lookup
failed immediately and the whole invocation - including the fan-out to the
halves - never started. Not a "no LEDs on the dongle" problem so much as a
"the dongle never got as far as asking who has LEDs" problem.

Fix: enabled `CONFIG_ZMK_RGB_UNDERGLOW=y` + `CONFIG_SPI=y` on
`skeletyl_dongle.conf`, plus a stub `led_strip` (`worldsemi,ws2812-spi`,
`chain-length = <1>`) on `&spi1` in `skeletyl_dongle.overlay` - that bus is
already enabled with default pinctrl (MOSI on P0.10) on nice!nano's own
board files and otherwise unused here, so no new pin claims needed.
`rgb_underglow.c` hard-`#error`s at compile time without a `zmk,underglow`
chosen node, so the stub isn't optional once the Kconfig is on - nothing is
physically connected to it, the pixel count is arbitrary and never matters,
it exists purely so the central-side `&rgb_ug` device resolves. Both
halves' own `CONFIG_ZMK_RGB_UNDERGLOW` + real `led_strip` were untouched -
they already had everything needed on the receiving end.

### Added `&out OUT_BLE` output selector, wireless-capable now that a dongle exists

Previously only `&out OUT_USB` existed on `CMD`/`MIR` - no way to switch the
dongle's output to BLE.

- `CMD` layer: `R` (old lone `OUT_USB` slot) freed to `___`; `X`/`C` now
  `&out OUT_BLE` / `&out OUT_USB` - deliberately placed one column off the
  home position (needs `&mo CMD` held + a same-hand reach), so the output
  isn't switched by accident.
- `MIR` layer: mirrored on the right hand, `M`/`,` = `&out OUT_USB` /
  `&out OUT_BLE` - kept on `MIR` rather than `CMD`'s own right half, so the
  same "awkward, deliberate reach" property holds on both hands. `MIR`'s
  pre-existing lone `OUT_USB` on `U` is untouched.

Confirmed working on hardware: dongle switches to `OUT_BLE` and pairs
directly to the host over Bluetooth, same as charybdis.

### ALT layer: `.`/`/` switched from `[`/`]` to `{`/`}`

Ported from the same fix on `charybdis`. `_ RA(DOT)`/`_ RA(FSLH)` (giving
`[`/`]` under Universal Layout's AltGr mapping) changed to
`_ LS(RA(DOT))`/`_ LS(RA(FSLH))` for `{`/`}`. `LS(RA(DOT))` = `{` verified
via `m_elmap`'s `%{` output; `LS(RA(FSLH))` = `}` follows the same pattern,
confirmed working on charybdis's hardware, not yet flashed here.
