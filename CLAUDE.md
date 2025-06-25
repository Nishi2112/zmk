# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ZMK Overview

ZMK (Zephyr Mechanical Keyboard) is an open-source keyboard firmware built on the Zephyr RTOS. It uses an event-driven architecture with a modular behavior system for keyboard actions, designed for wireless keyboards with excellent battery life.

## Environment Setup

### Prerequisites (Ubuntu/Debian)
```bash
# Install dependencies
sudo apt update
sudo apt install -y git wget python3-pip cmake ninja-build gperf ccache dfu-util \
  device-tree-compiler python3-dev python3-setuptools python3-tk python3-wheel \
  xz-utils file make gcc gcc-multilib g++-multilib libsdl2-dev

# Install West
pip3 install --user -U west

# Initialize West workspace (if not already done)
west init -l app/
west update
```

### Docker Alternative
```bash
docker pull docker.io/zmkfirmware/zmk-build-arm:3.5
```

## Build Commands

```bash
# Basic build
west build -s app -b <board> -- -DSHIELD=<shield>

# Clean build (recommended for first build or after major changes)
west build -s app -b <board> -- -DSHIELD=<shield> -p

# Common board examples
west build -s app -b nice_nano_v2 -- -DSHIELD=corne_left
west build -s app -b seeeduino_xiao_rp2040 -- -DSHIELD=xiao_test
west build -s app -b nrfmicro_13 -- -DSHIELD=kyria_left

# Flash firmware
west flash

# Build UF2 file for drag-and-drop flashing
west build -s app -b nice_nano_v2 -- -DSHIELD=corne_left -DBUILD_OUTPUT_UF2=y
```

## Common Build Errors and Solutions

### 1. West Not Found
```bash
# Add to PATH
echo 'export PATH=~/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 2. CMake Configuration Errors
```bash
# Clear build directory
rm -rf build/

# Ensure west is updated
west update

# Try pristine build
west build -s app -b <board> -- -DSHIELD=<shield> -p
```

### 3. Memory Overflow (common on nRF52840)
```conf
# Add to <shield>.conf
CONFIG_ZMK_BLE=n  # Disable BLE if not needed
CONFIG_ZMK_USB_LOGGING=n  # Disable USB logging
CONFIG_LOG=n  # Disable all logging
```

### 4. Shield Not Found
```bash
# List available shields
ls app/boards/shields/

# Ensure shield name matches directory name exactly
```

## Testing Commands

```bash
# Run all tests
./app/run-test.sh all

# Run specific test with snapshot update
ZMK_TESTS_AUTO_ACCEPT=1 ./app/run-test.sh tests/hold-tap/balanced/1-dn-up

# Verbose test output
ZMK_TESTS_VERBOSE=1 ./app/run-test.sh tests/<test_name>

# Run with specific board
./app/run-test.sh tests/<test_name> -- -DBOARD=native_posix

# BLE tests
./app/run-ble-test.sh <test_name>
```

## Debugging Techniques

### 1. USB Logging
```bash
# Build with USB logging
west build -s app -b <board> -- -DSHIELD=<shield> \
  -DZMK_EXTRA_MODULES="app/snippets/zmk-usb-logging"

# View logs (Linux)
screen /dev/ttyACM0 115200

# View logs (macOS)
screen /dev/tty.usbmodem* 115200

# Exit screen: Ctrl+A, then K
```

### 2. Add Debug Logging to Code
```c
#include <zephyr/logging/log.h>
LOG_MODULE_DECLARE(zmk, CONFIG_ZMK_LOG_LEVEL);

// In your function
LOG_DBG("Debug message: value=%d", some_value);
LOG_INF("Info message");
LOG_ERR("Error occurred: %d", error_code);
```

### 3. Debug Build Configuration
```conf
# Add to <shield>.conf
CONFIG_ZMK_USB_LOGGING=y
CONFIG_LOG=y
CONFIG_ZMK_LOG_LEVEL_DBG=y
CONFIG_LOG_BUFFER_SIZE=4096
CONFIG_LOG_PROCESS_THREAD_STACK_SIZE=2048
```

## Keymap Development

### Basic Keymap Structure
```dts
// app/boards/shields/<shield>/<shield>.keymap
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>

/ {
    keymap {
        compatible = "zmk,keymap";
        
        default_layer {
            bindings = <
                &kp Q  &kp W  &kp E  &kp R  &kp T
                &kp A  &kp S  &kp D  &kp F  &kp G
                &kp Z  &kp X  &kp C  &kp V  &kp B
            >;
        };
        
        raise_layer {
            bindings = <
                &kp N1  &kp N2  &kp N3  &kp N4  &kp N5
                &trans  &trans  &trans  &trans  &trans
                &bt BT_CLR  &bt BT_SEL 0  &bt BT_SEL 1  &trans  &trans
            >;
        };
    };
};
```

### Advanced Features

#### Combos
```dts
/ {
    combos {
        compatible = "zmk,combos";
        combo_esc {
            timeout-ms = <50>;
            key-positions = <0 1>;  // Q+W
            bindings = <&kp ESC>;
        };
    };
};
```

#### Macros
```dts
/ {
    macros {
        email_macro: email_macro {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp U &kp S &kp E &kp R &kp AT &kp E &kp X &kp A &kp M &kp P &kp L &kp E>;
        };
    };
};
```

#### Hold-Tap (Mod-Tap)
```dts
&mt {
    tapping-term-ms = <200>;
};

// Usage: hold for shift, tap for A
&mt LSHFT A
```

## Common Behaviors Reference

| Behavior | Binding | Description | Example |
|----------|---------|-------------|---------|
| Key Press | `&kp` | Single key press | `&kp A` |
| Layer | `&mo` | Momentary layer | `&mo 1` |
| Layer Toggle | `&tog` | Toggle layer | `&tog 2` |
| Bluetooth | `&bt` | BT commands | `&bt BT_CLR` |
| Reset | `&reset` | Reset device | `&reset` |
| Bootloader | `&bootloader` | Enter bootloader | `&bootloader` |
| Hold-Tap | `&mt` | Mod-tap | `&mt LSHFT A` |
| Macro | `&macro_name` | Custom macro | `&email_macro` |
| None | `&none` | No action | `&none` |
| Transparent | `&trans` | Pass to lower layer | `&trans` |

## Bluetooth Configuration

### Basic BLE Settings
```conf
# app/boards/shields/<shield>/<shield>.conf
CONFIG_ZMK_BLE=y
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y  # Increase TX power
CONFIG_ZMK_BLE_PASSKEY_ENTRY=y  # Enable passkey
CONFIG_BT_MAX_CONN=5  # Max connections
CONFIG_BT_MAX_PAIRED=5  # Max paired devices
```

### Common BLE Issues

#### Connection Drops
```conf
# Increase supervision timeout
CONFIG_BT_PERIPHERAL_PREF_MIN_INT=6
CONFIG_BT_PERIPHERAL_PREF_MAX_INT=12
CONFIG_BT_PERIPHERAL_PREF_LATENCY=30
CONFIG_BT_PERIPHERAL_PREF_TIMEOUT=400
```

#### Pairing Issues
```dts
// Clear all bonds
&bt BT_CLR

// Select profile
&bt BT_SEL 0  // Select profile 0
&bt BT_SEL 1  // Select profile 1
```

#### Power Optimization
```conf
CONFIG_ZMK_IDLE_TIMEOUT=30000  # 30 seconds
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000  # 15 minutes
```

## Performance Optimization

### Memory Optimization
```conf
# Reduce logging
CONFIG_LOG=n
CONFIG_ZMK_USB_LOGGING=n

# Reduce features
CONFIG_ZMK_DISPLAY=n  # Disable display if not used
CONFIG_ZMK_RGB_UNDERGLOW=n  # Disable RGB if not used

# Optimize stack sizes
CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=2048
CONFIG_MAIN_STACK_SIZE=2048
```

### Build Size Optimization
```conf
CONFIG_SIZE_OPTIMIZATIONS=y
CONFIG_LTO=y  # Link-time optimization
```

## Architecture Deep Dive

### Event Flow
```
Physical Key Press → GPIO Interrupt → Position State Changed Event → 
Key Position State Changed Listener → Behavior Processing → 
HID Report → USB/BLE Transmission
```

### Key File Locations
- Behaviors: `app/src/behaviors/` (implementations)
- Events: `app/include/zmk/events/` (definitions)
- HID: `app/src/hid.c` and `app/include/zmk/hid.h`
- BLE: `app/src/ble/` (all BLE functionality)
- USB: `app/src/usb*` files
- Split: `app/src/split/` (split keyboard support)

### Adding a New Behavior
1. Define behavior in `app/dts/behaviors/`
2. Implement in `app/src/behaviors/behavior_<name>.c`
3. Add to `app/src/behaviors/CMakeLists.txt`
4. Define binding in `app/dts/bindings/behaviors/`

## Split Keyboard Configuration

### Basic Split Setup
```conf
# Left half: <shield>_left.conf
CONFIG_ZMK_SPLIT=y
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Right half: <shield>_right.conf  
CONFIG_ZMK_SPLIT=y
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
```

### Transport Selection
```conf
# BLE transport (default)
CONFIG_ZMK_SPLIT_BLE=y

# Serial transport
CONFIG_ZMK_SPLIT_BLE=n
CONFIG_ZMK_SPLIT_SERIAL=y
```

## Real-World Examples

### Example 1: Home Row Mods (from Cradio)
```dts
&mt {
    flavor = "tap-preferred";
    tapping-term-ms = <220>;
    quick-tap-ms = <150>;
    require-prior-idle-ms = <100>;
    hold-trigger-key-positions = <0 1 2 3 4 5>;
};
```

### Example 2: Encoder Support
```dts
&encoder {
    status = "okay";
    a-gpios = <&pro_micro 21 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;
    b-gpios = <&pro_micro 20 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;
    steps = <80>;
};

/ {
    sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN>;
};
```

## Important Development Notes

- Device tree changes require clean builds (`-p` flag)
- Always check existing shields for patterns before creating new ones
- Test on `native_posix` board first for faster iteration
- Use `ZMK_CONFIG` env var to point to your config directory
- Memory constraints are real on nRF52840 - optimize aggressively
- BLE and USB can coexist but increase memory usage significantly
- Split keyboards always build separately for each half

## Useful Commands Reference

```bash
# Clean everything
rm -rf build/
west update

# List USB devices
ls /dev/tty*

# Monitor power consumption
west build -s app -b nice_nano_v2 -- -DSHIELD=corne_left -DCONFIG_ZMK_BATTERY_REPORTING=y

# Generate compilation database
west build -s app -b nice_nano_v2 -- -DSHIELD=corne_left -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```