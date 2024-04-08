# Pokemon Go Spoofing

## for MOTO G4, Android 8

1. install `adb-setup`, version >=1.4.3; this gives `fastboot.exe`, `adb.exe`
2. unlock bootloader.
3. in `developer options` (tap `About phone` > `Build number` multiple times), enable
    1. `OEM unlocking`
    2. `USB debugging`
    3. `Select USB configuration` = MTP (or File transfer)
    4. `Advanced reboot` (not required, but helpful when flashing ROM is needed)
4. install magisk
    1. `adb reboot bootloader` Reboots your Android device into fastboot / bootloader mode.
    2. `fastboot devices` to see devices
    3. `fastboot boot twrp-<version>.img` to load twrp recovery (find img here: https://twrp.me/Devices/)
    4. copy `Magisk-v24.3.zip` or .apk from PC to device (https://github.com/topjohnwu/Magisk/releases)
    5. in twrp, `install`, select magisk apk, use default options, install & reboot
    6. open Magisk, settings > hide magisk app
    7. download `root checker` from Google Play, to check root
    8. download `SafetyNet Test` (by BITS Apps) from Google Play, to check network
5. troubleshooting: SafetyNet Test failed response payload validation:
    1. enable Zygisk in the Magisk app's options, do the same for the Denylist
    2. configure the Denylist with the apps you wish to hide root from (e.g. pokemon go)
    3. download `Universal Safetynet Fix` https://github.com/kdrag0n/safetynet-fix/releases
    4. install it as a module and reboot
6. add Pokemon Go to Magisk's denylist
7. download `Fake GPS Joystick & Routes Go`
8. install SmaliPatcher:
    1. download smali patcher, 
        1. https://forum.xda-developers.com/t/module-smali-patcher-7-4.3680053/
        2. https://forum.xda-developers.com/m/fomey.1620550/#recent-content
    3. run it with admin permission.
    4. wait till the status says `idle`
    5. click `ADB PATCH`
    6. wait, a `SmaliPatcherModule-....zip` is created on PC
    7. copy the zip file to device
    8. open magisk, go to Modules, install the zip file
    9. reboot

### Unlock bootloader

1. find articles online, it differs on each device.

### Flash ROM

1. `adb reboot bootloader` Reboots your Android device into fastboot / bootloader mode.
2. `fastboot devices` to see devices
3. `fastboot boot twrp-<version>.img` to load twrp recovery
4. use TWRP `Advanced > ADB sideload`
5. on PC, run `adb sideload <name of rom>.zip`
6. expected error: `(47%) adb: failed to read command: No error`, it's not an error, continue
7. reboot device

---

## for OnePlus 6, Android 11

link: [source](https://docs.google.com/document/d/1tc1ygrT2q51jSmFMy6-VsRmc-lsROxrFEcAtrqH7z28/edit);
from: [reddit](https://www.reddit.com/r/PoGoAndroidSpoofing/comments/171ry0n/rooted_method_1_lsposed_guide_for_android_8_9_10/);
from: [reddit](https://www.reddit.com/r/PoGoAndroidSpoofing/comments/rtyeyg/clickpress_here_mega_post_4_everything_you_need/)

1. install `Magisk`, enable `Zygisk`
2. add `LSposed` to `Magisk`
3. enable `Hide Mock Location` in `LSposed`
4. turn off Google location accuracy
5. install joystick app
6. add `SafetyNet` to `Magisk`

## for Redmi, Android 10

1. download ROM: https://xiaomirom.com
2. download MiFlash tool: https://xiaomiflashtool.com/ (or from other sources)
3. flash the ROM

---

## Locations

- Singapore:
  - 16:00 - 17:00
    - Maxwell
    - Chinatown
    - Bugis
    - Anglo-Chinese school (Saturday)
    - Dhoby Ghaut
- New York:
  - Central park (all day)
