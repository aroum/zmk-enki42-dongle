# ZMK Config for Corne-Like Keyboards with Dongle

This repository provides a custom ZMK configuration for Corne-like wireless keyboards (e.g., Enki42) using a **three-controller dongle architecture**. By offloading the central BLE host role to a dedicated USB dongle, battery efficiency for both keyboard halves is drastically improved.

As a dongle, you can use any board based on the nRF52840 chip—such as an original nice!nano v2 or its clones, a Seeed Studio XIAO BLE, or compact off-the-shelf USB dongles from Holyiot. Using third-party hardware usually requires only minor pin mapping and configuration adjustments for that specific board.

## Overview & Theoretical Background

### How ZMK Dongle Architecture Works (BLE)

In a standard ZMK wireless split setup:

- **Left/Primary half:** Acts as the BLE **Central** (Master). It receives signals from the peripheral half, processes keymaps/behaviors, and maintains the BLE connection to the host PC or device.
    
- **Right/Secondary half:** Acts as a BLE **Peripheral** (Slave).
    

Because the primary half manages two active wireless links (keyboard-to-keyboard and keyboard-to-host), it drains its battery significantly faster (often lasting 1–2 weeks vs. several weeks/months for the peripheral half).

#### The Dongle Architecture

In this repository's configuration:

1. **Dongle:** Acts as the BLE Central (Master). It collects input from all peripherals, processes keymaps, and outputs HID keycodes to the host PC via USB HID or BLE (Bluetooth).
    
2. **Left Half:** Acts as a BLE **Peripheral (Slave)**.
    
3. **Right Half:** Acts as a BLE **Peripheral (Slave)**.
    
4. _(Optional)_ **Additional Peripherals:** E.g., an autonomous trackball, touchpad, or macropad as BLE **Peripherals (Slaves)**.
    

```
┌─────────────────┐       BLE        ┌────────────────┐
│  Left Half      ├─────────────────►│                │
│ (Peripheral)    │                  │     Dongle     │──── USB HID / BLE ────► Host PC
└─────────────────┘                  │   (Central)    │
┌─────────────────┐       BLE        │                │
│  Right Half     ├─────────────────►│                │
│ (Peripheral)    │                  │                │
└─────────────────┘                  │                │ 
┌─────────────────┐       BLE        │                │
│ Trackball/Pad   ├─────────────────►│                │
│ (Peripheral)    │                  │                │
└─────────────────┘                  └────────────────┘
```

#### Key Advantages

- **Dramatically Increased Battery Life:** Both physical keyboard halves operate purely in peripheral low-power mode, multiplying runtime (weeks/months instead of days).
    
- **Even Battery Wear:** Both halves consume power at virtually identical rates.
    
- **Multi-Peripheral Support:** Because the dongle is the Central, ZMK allows connecting **more than two peripherals** simultaneously (e.g., Left half, Right half, an autonomous trackball, or a macropad).
    
    - Controlled via `CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS` (e.g., set to `2`, `3`, or more).
        
    - Total BLE connections supported by the dongle are configured using `CONFIG_BT_MAX_CONN` (calculated as: `number of peripherals` + `number of saved host BT profiles`).
        
- **Reduced Latency:** Direct, un-routed connection from both halves straight to the central dongle.
    
- **Reversibility:** You can revert to a standard two-controller wireless setup at any time by reflashing standard ZMK split firmware.
    

#### Trade-offs & Workarounds

- **Loss of Direct USB HID on Keyboard Halves:** Plugs on the halves are used for charging only. Typing via USB requires connecting the dongle.
    
- **Multiple Device Switching:** Standard ZMK BT profile switching commands target the Central device. Since the dongle is the Central:
    
    - _Workaround:_ If you power the dongle via an external source (e.g., powerbank, 18650 battery cell, or powered hub), you can still pair the dongle via Bluetooth to multiple host devices and use standard ZMK Bluetooth profile switching.
        
- **Telemetry Limitations:** Standard split battery level reporting to the host OS may require specific ZMK feature flags or custom modules such as [zmk-battery-center](https://github.com/kot149/zmk-battery-center).
    

### Deep Dive: ZMK Protocols & Dongle Architecture (BLE vs. ESB vs. Keychron)

To understand wireless dongles in custom keyboards, it is important to differentiate between standard ZMK BLE, custom ESB transport modules, and commercial 2.4GHz implementations like Keychron's.

#### 1. Native ZMK BLE Dongle (This Repository)

- **Protocol:** Standard Bluetooth Low Energy (BLE).
    
- **Architecture:** Dongle is the **Master/Central**, both keyboard halves (and optional extra peripherals) are **Slave/Peripherals**.
    
- **Pros:** 100% open-source, uses standard ZMK drivers, works out-of-the-box on any supported nRF52840 hardware (nice!nano, XIAO BLE, etc.).
    

#### 2. Nordic ESB Split Transport ([`badjeff/zmk-feature-split-esb`](https://github.com/badjeff/zmk-feature-split-esb "null"))

- **Protocol:** Nordic Enhanced ShockBurst (ESB), a lightweight 2.4GHz proprietary protocol with ultra-low latency and low overhead compared to BLE.
    
- **Architecture:** Same as ZMK BLE dongles—the **dongle remains the Master/Central**, while the keyboard halves remain Slaves/Peripherals.
    
- **Trade-off:** Because the keyboard halves remain slave devices, you still cannot plug a USB cable into the keyboard half and expect it to automatically become a direct USB HID device; all input processing remains tied to the dongle master.
    

#### 3. Keychron 2.4G RF Implementation ([`Keychron/zmk`](https://github.com/Keychron/zmk "null"))

- **Protocol:** Nordic ESB using a precompiled static binary ([`lib_nrf_esb_24G.a`](https://github.com/Keychron/zmk/blob/keychron_bpro/app/src/24G/lib_nrf_esb_24G.a "null")).
    
- **Architecture:** Reversed! **The Keyboard is the Master/Central**, and the dongle acts purely as a passive RF receiver.
    
- **How it works:** Keychron customizes ZMK's output behaviors ([`behavior_outputs.c`](https://github.com/Keychron/zmk/blob/f44436b31033c81df6ab07e37178e6678d5aeace/app/src/behaviors/behavior_outputs.c#L81)) to dynamically switch output channels on the keyboard itself between `USB`, `Bluetooth`, or `2.4G Dongle`.
    
- **Why we can't easily port it:** Keychron's dongle firmware and RF receiver schematics are closed-source. Without access to the receiver's companion firmware, this architecture cannot be compiled onto generic DIY controllers.
    

#### Comparison Matrix

| **Feature** | **Native ZMK BLE (This Repo)** | **ZMK ESB Module (badjeff)** | **Keychron 2.4G (Keychron/zmk)** |

| **Transport** | Standard BLE | Nordic ESB (2.4GHz) | Nordic ESB (2.4GHz) |

| **Master Role** | Dongle | Dongle | **Keyboard** |

| **Output Switching** | On Dongle | On Dongle | On Keyboard (`USB / BT / 2.4G`) |

| **Plug USB to Keyboard** | Charging only | Charging only | Direct USB HID output |

| **Open Source Status** | 100% Open Source | 100% Open Source | Closed binary (`lib_nrf_esb_24G.a`) |

| **Hardware** | Any nRF52840 (n!n, XIAO) | Any nRF52840 (n!n, XIAO) | Proprietary Keychron hardware/dongle |

## Hardware Compatibility

The dongle can be built using any supported nRF52840 board (or other ZMK-compatible controllers):

- **nice!nano (n!n)**
    
- **Seeed Studio XIAO BLE** (see dedicated branch in this repo)
    
- **nRFmicro**
    
- **Standalone USB nRF52840 Dongles** (e.g., generic AliExpress nRF52840 USB sticks with minor bootloader/firmware adjustments)
    

## Repository Features

- Configured specifically for [**Enki 42**](https://www.reddit.com/r/ErgoMechKeyboards/comments/qeq2qg/enki42_slim_ergo_keyboard/), but easily adaptable to **Corne (crkbd)** or any split keyboard.
    
- Includes [**Watchman's layout**](https://github.com/aroum/Watchman-layouts).
    
- Multiple branches available for different dongle controller types (e.g., XIAO BLE branch).
    

To customize the device name, update line `CONFIG_ZMK_KEYBOARD_NAME="Enki42"` in `config/boards/shields/enki42/enki42.conf`.

## Installation & Flashing

1. **Reset Settings:** Flash the [ZMK Settings Reset Firmware](https://zmk.dev/docs/troubleshooting#split-keyboard-halves-unable-to-pair) onto all **three** controllers (Dongle, Left Half, Right Half) to clear prior pairing keys.
    
2. **Flash Firmware:** Flash the corresponding firmware binaries onto each of the three devices:
    
    - Dongle firmware -> Dongle board
        
    - Left half firmware -> Left keyboard board
        
    - Right half firmware -> Right keyboard board
        
3. **Pairing:** Power on the dongle first, then the halves. If they do not connect automatically, press the physical reset buttons on the dongle and keyboard halves simultaneously.
    

## Credits & References

- [**@slicemk**](https://github.com/slicemk "null") for the original split dongle implementation concept ([SliceMK Split Dongle Documentation](https://www.slicemk.com/pages/split-dongle)).
    
- [**@badjeff**](https://github.com/badjeff) for the [`zmk-feature-split-esb`](https://github.com/badjeff/zmk-feature-split-esb) module.
    
- [**Keychron ZMK Repository**](https://github.com/Keychron/zmk) for 2.4G ESB source inspection.
    
- [**ZMK Documentation**](https://zmk.dev/docs/development/hardware-integration/dongle#adding-a-dongle) on dongle integration.
    
- [**@benvallack**](https://github.com/benvallack/zmk-config-card) for card keyboard reference configs.
    
- [**@krikun98**](https://github.com/krikun98/) for configuration fixes and setup assistance.
