# Narfpad ZMK Configuration - Hardware Setup

## Overview

This is a ZMK firmware configuration for a **narfpad macropad** running on hardware originally designed for the **narfbloq** keyboard.

## Hardware Components

| Component | Description |
|-----------|-------------|
| **PCB** | [Narfpad Macropad](https://keebd.com/products/narfpad-macropad-keyboard-kit) from KEEBD |
| **Controller** | nRF52840 chip (Seeed XIAO BLE form factor, from narfbloq design) |
| **Layout** | 10-key macropad with rotary encoder |
| **Power** | USB-only (no battery currently) |

## Key Repositories

- **Narfpad original firmware**: https://github.com/sebastian-stumpf/narfpad
- **Narfbloq (controller source)**: https://github.com/sebastian-stumpf/narfbloq

## ⚠️ CRITICAL: PCB-to-Chip Alignment

**This setup uses a chip designed for the narfbloq on the narfpad PCB.**

The GPIO pin mappings in the overlay file have been carefully aligned to ensure the narfbloq controller works correctly with the narfpad matrix. The current configuration is **verified working** - do not modify the overlay pin mappings unless you understand both hardware designs.

### Current Pin Mapping (Working)

**Matrix (3 rows × 4 columns):**
- Row GPIOs: `xiao_d 4`, `xiao_d 5`, `xiao_d 6`
- Column GPIOs: `xiao_d 3`, `xiao_d 2`, `xiao_d 1`, `xiao_d 0`
- Diode direction: `col2row`

**Rotary Encoder (EC11):**
- A pin: `xiao_d 7`
- B pin: `xiao_d 8`
- Steps: 80 (triggers-per-rotation: 20)

## Build Configuration

### Build Target (`build.yaml`)

```yaml
include:
  - board: xiao_ble
    shield: narfpad
```

### Runtime Config

**Board Config (`config/xiao_ble.conf`)** - Controller capabilities:
- **USB**: Enabled (`CONFIG_ZMK_USB=y`)
- **Bluetooth**: Disabled (`CONFIG_BT=n`) - USB-only to save power
- **Logging**: Disabled (`CONFIG_ZMK_USB_LOGGING=n`)

**Shield Config (`narfpad.conf`)** - PCB hardware:
- **Encoder**: EC11 with global thread trigger (connected to narfpad PCB)

> ⚠️ **Important**: Bluetooth must be disabled in the **board** config file (`xiao_ble.conf`), not the shield config. Shield configs cannot override board-level Kconfig options like `CONFIG_BT`.

## Current Keymap

The macropad is configured for **VS Code / Cursor IDE** workflows:

| Layer | Purpose |
|-------|---------|
| DEFAULT | IDE shortcuts (close tab, command palette, save, find, references, etc.) |
| LOWER | System layer (Bluetooth controls, reset, bootloader) |

Encoder actions:
- DEFAULT: Cycle through editor tabs (`Cmd+Opt+Left/Right`)
- LOWER: Volume control

## Building Firmware

Firmware is built via **GitHub Actions**. Push to the repository triggers the ZMK build workflow.

## Future: Adding Bluetooth/Battery

Currently running USB-only (no battery). To enable wireless operation later:

1. **Connect a battery** to the XIAO BLE's battery pads (3.7V LiPo)

2. **Update `config/xiao_ble.conf`**:
   ```
   # Remove or comment out:
   # CONFIG_BT=n

   # Add these for wireless:
   CONFIG_ZMK_BLE=y
   CONFIG_BT_CTLR_TX_PWR_PLUS_8=y  # Max Bluetooth range

   # Power management for battery life:
   CONFIG_ZMK_SLEEP=y
   CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000  # Sleep after 15min idle
   ```

3. **Keep USB as fallback**:
   ```
   CONFIG_ZMK_USB=y  # Keep USB enabled for charging + wired mode
   ```

The LOWER layer already has `&bt BT_PRV`, `&bt BT_CLR`, `&bt BT_NXT` bindings ready for pairing.

## Troubleshooting

If keys don't work after hardware changes:
1. Verify physical connections between controller and PCB
2. Check that the controller orientation matches the expected pin-out
3. Do **not** modify the overlay file unless you've mapped the physical pins correctly
4. The current overlay is confirmed working - use it as reference

## File Structure

```
config/
├── xiao_ble.conf                   # Board-level config (BT disable goes here!)
├── boards/
│   └── shields/
│       └── narfpad/
│           ├── Kconfig.defconfig   # Keyboard name config
│           ├── Kconfig.shield      # Shield detection
│           ├── narfpad.conf        # Shield-specific settings
│           ├── narfpad.keymap      # Key bindings
│           ├── narfpad.overlay     # Hardware pin definitions ⚠️
│           └── narfpad.zmk.yml     # ZMK metadata
└── west.yml                        # ZMK module config
```

---

*Last verified: January 2026*
