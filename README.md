# ZMK config for corne-like keyboard with dongle

The config allows you to use 3 n!n, one for the dongle, one for the left half, one for the right half. Using a dongle allows you to significantly increase the time of the keyboard, but at the same time you can not switch between different devices as with bluetooth. More details on the [slicemk page](https://www.slicemk.com/pages/split-dongle).

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

## Credit


* [@slicemk](https://github.com/slicemk) for an original idea
 
* [@benvallack](https://github.com/benvallack/zmk-config-card) card keyboard config as a reference 

* [@krikun98](https://github.com/krikun98/) for config fixes and other help
