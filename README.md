# Scanner Pocket

A compact ZMK-based keyboard status display using Sharp Memory LCD.

[![Build](https://github.com/t-ogura/zmk-scanner-pocket/actions/workflows/build.yaml/badge.svg)](https://github.com/t-ogura/zmk-scanner-pocket/actions/workflows/build.yaml)

## Overview

Scanner Pocket is a standalone display device that shows real-time keyboard status via BLE Advertisement scanning. It monitors your ZMK keyboards without using a BLE connection slot.

### Features

- **Non-intrusive monitoring**: Keyboards maintain full 5-device BLE connectivity
- **Multi-keyboard support**: Monitor up to 3 keyboards simultaneously
- **Low power**: Sharp Memory LCD consumes only ~60µW
- **Real-time display**: Layer, battery, modifiers, WPM, BLE profile status

## Hardware

| Component | Specification |
|-----------|---------------|
| MCU | Seeed Studio XIAO nRF52840 |
| Display | Sharp Memory LCD LS013B7DH05 (144x168, 1-bit) |
| Orientation | Landscape (168x144) |

### Pinout

| Signal | XIAO Pin |
|--------|----------|
| LCD_SI (MOSI) | D10 |
| LCD_SCLK | D8 |
| LCD_SCS | D7 |
| DISP | D3 |
| EXTCOMIN | D6 |

## Display Layout

```
┌────────────────────────────────────┐
│                         [▓▓▓▓]▌ 85│ Scanner Battery
│         Device Name                │
│ WPM                          BLE   │
│  42                           0    │
│           Layer                    │
│          0 1 [2] 3 4               │
│         󰘴 󰘶 󰘵                      │ Modifiers
│                                    │
│   L 85        R 42                 │ Keyboard Battery
│   [████████]  [████    ]           │
└────────────────────────────────────┘
```

## Building

### GitHub Actions (Recommended)

Push to the repository to trigger automatic builds. Download the firmware from the Actions artifacts.

### Local Build

```bash
# Clone repository
git clone https://github.com/t-ogura/zmk-scanner-pocket.git
cd zmk-scanner-pocket

# Setup virtual environment
python3 -m venv .venv
source .venv/bin/activate
pip install west

# Initialize and update
west init -l config
west update

# Build
west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=scanner_pocket \
  -DZMK_CONFIG="$(pwd)/config"
```

Output: `build/zephyr/zmk.uf2`

## Flashing

1. Connect XIAO nRF52840 via USB
2. Double-tap reset button to enter bootloader (LED fades)
3. Copy `zmk.uf2` to the mounted drive

## Configuration

Edit `config/scanner_pocket.conf` to customize:

```conf
# Number of keyboards to monitor
CONFIG_PROSPECTOR_MAX_KEYBOARDS=3

# Maximum layers to display
CONFIG_PROSPECTOR_MAX_LAYERS=16

# Enable debug logging
CONFIG_LOG=y
CONFIG_LOG_DEFAULT_LEVEL=3
```

## Keyboard Compatibility

Your keyboard needs [Prospector ZMK Module](https://github.com/t-ogura/prospector-zmk-module) with status advertisement enabled:

```conf
# In your keyboard's .conf
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="YourKeyboard"
```

## License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for details.

### Third-Party Licenses

| Component | License | Source |
|-----------|---------|--------|
| ZMK Firmware | MIT | [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk) |
| Prospector Module | MIT | [t-ogura/prospector-zmk-module](https://github.com/t-ogura/prospector-zmk-module) |
| QuinqueFive Font | SIL OFL 1.1 | [GGBotNet](https://ggbot.itch.io/quinquefive-font) |
| Unscii Font | Public Domain | [Viznut](http://viznut.fi/unscii/) |
| NerdFonts | MIT | [YADS](https://github.com/janpfischer/yads) |

## Credits

- [ZMK Firmware](https://zmk.dev/) - The keyboard firmware framework
- [YADS](https://github.com/janpfischer/yads) - NerdFont icons inspiration
- [GGBotNet](https://ggbot.itch.io/) - QuinqueFive pixel font
- [Viznut](http://viznut.fi/) - Unscii font

## Related Projects

- [prospector-zmk-module](https://github.com/t-ogura/prospector-zmk-module) - Core Prospector module
- [zmk-config-prospector](https://github.com/t-ogura/zmk-config-prospector) - Color display scanner config
