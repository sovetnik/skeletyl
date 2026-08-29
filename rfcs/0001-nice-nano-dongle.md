# 0001: nice!nano dongle + battery proxy, matching charybdis

## Context

skeletyl is currently a 2-piece split with no dongle: `skeletyl_left` is a
fully custom nRF52840 board (own `.dts`, kscan, USB) acting as BLE central,
`skeletyl_right` is the peripheral. Home-row mods have already replaced the
old `cm_gui/cm_alt/cm_ctrl/cm_sft` sticky-mod chain (branch `homerow`,
ported from `charybdis`).

The sibling project `charybdis` (xiao_ble dongle + nice_nano halves) went
through the same sticky-mod-to-homerow pivot earlier, plus added a dongle
and full central battery proxy (`CENTRAL_BATTERY_LEVEL_FETCHING`/`_PROXY`)
for both halves. skeletyl doesn't have a dongle, and its central
(`skeletyl_left`) does not currently proxy `skeletyl_right`'s battery to the
host - only its own.

Goal: bring skeletyl's topology and battery reporting in line with
charybdis's, for consistency across both personal keyboards and for the
power-bank/wireless-central use case charybdis already validated.

Key structural difference from charybdis: charybdis's halves and dongle are
all *shields* on top of vendor boards (`nice_nano`, `xiao_ble`), sharing one
`charybdis.dtsi`. skeletyl's halves are their own *boards* (bare nRF52840
SoC config, not a shield-on-socket model) - there is no existing
shield-common file to extend, and the new dongle can't just `#include` the
halves' board `.dtsi` across directories (same quote-include resolution
issue hit and fixed once already on charybdis).

Dongle MCU: **Adafruit nice!nano**, not xiao_ble (unlike charybdis) - this
is what's physically on hand for this build.

## Decision

1. **Relocate the keymap to `config/`.**
   `boards/arm/skeletyl/skeletyl.keymap` and `skeletyl_behaviors.dtsi` move
   to `config/skeletyl.keymap` / `config/skeletyl_behaviors.dtsi`. ZMK's
   `post_boards_shields.cmake` strips shield/board suffixes when searching
   for a keymap (`skeletyl_dongle` -> `skeletyl`, `skeletyl_left` ->
   `skeletyl`) and always searches `ZMK_CONFIG` (`config/`) first - so one
   file serves all three build targets, same pattern as charybdis's
   `config/charybdis.keymap`.

2. **New shield `boards/shields/skeletyl_dongle/`, targeting `nice_nano`.**
   `Kconfig.shield`, `Kconfig.defconfig` (`ZMK_SPLIT_ROLE_CENTRAL=y`,
   `ZMK_SPLIT=y`), `.conf` (`ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2` + battery
   fetching/proxy), `.overlay` with `zmk,kscan = &mock_kscan` (no physical
   matrix on the dongle itself).

3. **Duplicate the physical-layout node into the dongle shield**, rather
   than refactoring `skeletyl.dtsi` to share it. `skeletyl_5col_layout` /
   `default_transform` / `skeletyl_position_map` are ~40 lines of key
   coordinates; `skeletyl.dtsi` otherwise mixes in real board-level config
   (ADC, UART, flash partitions) that has no business being shield-visible.
   Lower risk than extracting a cross-directory shared include, at the cost
   of two copies of the coordinate table that must be kept in sync by hand
   if the physical layout ever changes.

4. **Flip `skeletyl_left`'s role to peripheral.**
   Remove `config ZMK_SPLIT_BLE_ROLE_CENTRAL default y` from
   `boards/arm/skeletyl/Kconfig.defconfig`'s `if BOARD_SKELETYL_LEFT` block.
   Both halves default to peripheral; the dongle shield claims central via
   its own `Kconfig.defconfig` (using the canonical `ZMK_SPLIT_ROLE_CENTRAL`
   name, not the legacy `_BLE_` alias `skeletyl_left` currently uses).

5. **`build.yaml`**: add `nice_nano/nrf52840/zmk` + `skeletyl_dongle` as the
   central build; keep `skeletyl_left`/`skeletyl_right` as peripheral
   builds (no more Studio/central cmake-args on `skeletyl_left`).

6. **Battery proxy on the dongle only**: `CENTRAL_BATTERY_LEVEL_FETCHING`/
   `_PROXY=y` in `skeletyl_dongle.conf`. No need for charybdis's bumped
   split-queue sizes (`POSITION_QUEUE_SIZE`/`SPLIT_RUN_QUEUE_SIZE`/stack) -
   those were sized for a trackball's continuous pointer-motion traffic;
   skeletyl has no pointing device, so ordinary keypress traffic is fine at
   ZMK's defaults.

## Consequences

- A `skeletyl_right_standalone_central` fallback build was added alongside
  the dongle target, mirroring charybdis's
  `charybdis_right_standalone_central`: `skeletyl_right` with
  `-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y` forced via cmake-args, for direct
  half-to-half BLE with no dongle present. Not yet confirmed on hardware.
- Moving the keymap into `config/` changes the file layout for the existing
  `skeletyl_left`/`skeletyl_right` builds too, even though their own board
  `.dtsi`/Kconfig/kscan wiring is untouched - low risk (same pattern already
  proven on charybdis) but worth a CI build check on all three targets
  before merging, not just the new dongle target.
- Physical-layout duplication is a known, accepted drag: if the physical
  keyboard's key coordinates ever change, both `skeletyl.dtsi` and
  `skeletyl_dongle`'s overlay need editing.
- No change to trackball/pointing-device config - skeletyl has none, unlike
  charybdis.
