# ZMK config for corne-like keyboard with dongle

The config allows you to use 3 n!n, one for the dongle, one for the left half, one for the right half. Using a dongle significantly increases battery life.. The dongle can be any device on the nRF52840 (or other supported ZMK controllers). For example, I made a separate branch in this repository where XIAO BLE is used as a dongle. More details on the [slicemk page](https://www.slicemk.com/pages/split-dongle).

Config contains [Watchman's layout](https://github.com/aroum/Watchman-layouts).

This is a config for [Enki 42](https://www.reddit.com/r/ErgoMechKeyboards/comments/qeq2qg/enki42_slim_ergo_keyboard/), but you can use it with corne or any other compatible keyboard.

You can change the name in line
```
CONFIG_ZMK_KEYBOARD_NAME="Enki42"
```
in file
```
config/boards/shields/enki42/enki42.conf
```
## Install

Before flashing this firmware, flash the [setting reset firmware](# ZMK config for corne-like keyboard with dongle

The config allows you to use 3 n!n, one for the dongle, one for the left half, one for the right half. Using a dongle significantly increases battery life.. The dongle can be any device on the nRF52840 (or other supported ZMK controllers). For example, I made a separate branch in this repository where XIAO BLE is used as a dongle. More details on the [slicemk page](https://www.slicemk.com/pages/split-dongle).

Config contains [Watchman's layout](https://github.com/aroum/Watchman-layouts).

This is a config for [Enki 42](https://www.reddit.com/r/ErgoMechKeyboards/comments/qeq2qg/enki42_slim_ergo_keyboard/), but you can use it with corne or any other compatible keyboard.

You can change the name in line
```
CONFIG_ZMK_KEYBOARD_NAME="Enki42"
```
in file
```
config/boards/shields/enki42/enki42.conf
```
## Install

Before flashing this firmware, flash the [setting reset firmware](https://zmk.dev/docs/troubleshooting#split-keyboard-halves-unable-to-pair) into all 3 controllers. Then flash this firmware on 3 devices. If the halves do not connect themselves, try pressing the reset buttons on the dongle and keyboards.

## Credit

* [@slicemk](https://github.com/slicemk) for an original idea
 
* [@benvallack](https://github.com/benvallack/zmk-config-card) card keyboard config as a reference 

* [@krikun98](https://github.com/krikun98/) for config fixes and other help
) into all 3 controllers. Then flash this firmware on 3 devices. If the halves do not connect themselves, try pressing the reset buttons on the dongle and keyboards.

## Credit

* [@slicemk](https://github.com/slicemk) for an original idea
 
* [@benvallack](https://github.com/benvallack/zmk-config-card) card keyboard config as a reference 

* [@krikun98](https://github.com/krikun98/) for config fixes and other help
