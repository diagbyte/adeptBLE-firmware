# AdeptBLE

ZMK firmware for **AdeptBLE**, a wireless trackball/pointing device built on a
Seeed Studio XIAO nRF52840 (`xiao_ble`) with a PixArt **PMW3610** optical sensor.

This repository is a [ZMK](https://zmk.dev) user-config module: it is built in
the cloud by GitHub Actions and produces a flashable `.uf2` firmware file.

> **Why this fork exists**
> Upstream ZMK `main` adopted the **Zephyr 4.1 / Hardware Model v2 (HWMv2)**
> update, which renamed boards and the PMW3610 driver bindings. Those breaking
> changes broke the GitHub Actions build (`Invalid BOARD` during *West Build*).
> This fork updates every reference to the new naming so the firmware builds
> again on current ZMK. See **[Compatibility changes](#compatibility-changes-zephyr-41--hwmv2)**.

---

## Hardware

| Part | Detail |
| --- | --- |
| Controller | Seeed Studio XIAO nRF52840 (`xiao_ble`, nRF52840) |
| Pointing sensor | PixArt PMW3610 trackball over SPI |
| Inputs | 6 direct-wired keys (`zmk,kscan-gpio-direct`) |
| Connectivity | Bluetooth LE + USB |
| Config tool | [ZMK Studio](https://zmk.dev/docs/features/studio) enabled |

## Features

- Mouse buttons (left / right / middle / MB4 / MB5) and click combos
- **Scroll** modes (momentary and "keep") via layer switching
- **Snipe** (precision / slow-cursor) mode
- Bluetooth profile management layer (`BT_SEL 0–4`, `BT_CLR`)
- Bootloader access layer for flashing
- Runtime keymap editing through ZMK Studio (`studio-rpc-usb-uart`)

## Layers

| # | Name | Purpose |
| --- | --- | --- |
| 0 | Default | Mouse buttons, click, layer access |
| 1 | Scroll-Momentary | Scroll while held |
| 2 | Scroll-Keep | Latched scroll mode |
| 3 | Device | Bootloader, clear BT bonds |
| 4 | Bluetooth | Select/clear BT profiles |
| 5 | Snipe | Reduced sensitivity (precision) |

---

## Building

The build runs automatically on every push via
[`.github/workflows/build.yml`](.github/workflows/build.yml), which calls ZMK's
reusable `build-user-config` workflow. The build matrix is defined in
[`build.yaml`](build.yaml).

To get firmware:

1. Push to this repository (or open a pull request).
2. Open the **Actions** tab and select the latest **Build ZMK firmware** run.
3. Download the `firmware` artifact and flash the `.uf2` to the XIAO in
   bootloader mode (double-tap reset, then copy the file to the USB drive).

### Dependencies

Defined in [`config/west.yml`](config/west.yml):

| Module | Source | Revision |
| --- | --- | --- |
| `zmk` | `zmkfirmware/zmk` | `main` |
| `zmk-pmw3610-driver` | `badjeff/zmk-pmw3610-driver` | `main` |

> **Note:** both modules track `main`. That keeps the fork current with upstream
> features (e.g. ZMK Studio) but also means an upstream breaking change can stop
> the build without warning — see the next section for the most recent example.
> If you prefer stability, pin these to a known-good tag/commit instead.

---

## Compatibility changes (Zephyr 4.1 / HWMv2)

ZMK's Zephyr 4.1 update (Hardware Model v2) requires every in-tree board to be
selected with a `zmk` variant and renamed the PMW3610 out-of-tree driver's
devicetree `compatible` string and Kconfig prefixes. The following changes were
applied to make this config build against current ZMK `main`:

| File | Before | After |
| --- | --- | --- |
| `build.yaml` | `board: seeeduino_xiao_ble` | `board: xiao_ble/nrf52840/zmk` |
| `boards/shields/AdeptBLE/AdeptBLE.zmk.yml` | `requires: [seeeduino_xiao_ble]` | `requires: [seeed_xiao]` |
| `boards/shields/AdeptBLE/AdeptBLE.overlay` | `compatible = "pixart,pmw3610";` | `compatible = "pixart,pmw3610-alt";` |
| `config/AdeptBLE.conf` | `CONFIG_PMW3610_*` | `CONFIG_PMW3610_ALT_*` |

Background:

- **Board target.** Under HWMv2 the old `seeeduino_xiao_ble` board no longer
  exists. The Seeed XIAO nRF52840 is now `xiao_ble`, and ZMK adds a `zmk`
  variant on the `nrf52840` SoC, giving the build target
  `xiao_ble/nrf52840/zmk` (it can be shortened to `xiao_ble//zmk`).
- **Shield requirement.** Shield metadata now references the *interconnect* the
  board exposes (`seeed_xiao`) rather than a specific board name, so the shield
  stays compatible with any seeed_xiao-based board.
- **PMW3610 driver.** The [badjeff PMW3610 driver](https://github.com/badjeff/zmk-pmw3610-driver)
  `main` branch (for ZMK 0.4 / Zephyr 4.1) renamed its compatible string to
  `pixart,pmw3610-alt` and all Kconfig symbols to `CONFIG_PMW3610_ALT_*` to
  avoid clashing with the PMW3610 driver now shipped inside ZMK/Zephyr.
- The `&xiao_d` / `&xiao_serial` device-tree node labels are still provided by
  the `seeed_xiao` interconnect and did not need changes.

---

## Repository structure

```
.github/workflows/build.yml          GitHub Actions build entry point
build.yaml                           Build matrix (board + shield + snippet)
config/
  west.yml                           ZMK + module dependencies
  AdeptBLE.conf                      Kconfig options (PMW3610, BLE, Studio, ...)
  AdeptBLE.keymap                    Keymap, layers, behaviors, combos
boards/shields/AdeptBLE/
  AdeptBLE.overlay                   Device tree: kscan, SPI, trackball, layout
  AdeptBLE.zmk.yml                   Shield metadata (Studio / web UI)
  Kconfig.shield / Kconfig.defconfig Shield Kconfig
zephyr/module.yml                    Marks this repo as a ZMK module
```

## Credits

- [ZMK Firmware](https://zmk.dev)
- [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)
