# Kinesis Advantage 360 Pro ZMK Config

## Modifying the keymap

[The ZMK documentation](https://zmk.dev/docs) covers both basic and advanced functionality and has a table of OS compatibility for keycodes. Please note that the RGB Underglow, Backlight and Power Management sections are not relevant to the Advantage 360 Pro's custom ZMK fork. For more information see [this note](#note)

There is a web based GUI available for editing the keymap. It is available at https://kinesiscorporation.github.io/Adv360-Pro-GUI. This repository is also compatible with certain other web based ZMK keymap editors however they may have keycodes or behaviours that are not implemented on the 360 Pro and could cause unusual behaviour or build failures. Furthermore changes made on other keymap editors may not be compatible if one goes back to using the Kinesis GUI.

Certain ZMK features (e.g. combos) require knowing the exact key positions in the matrix. They can be found in both image and text format [here](assets/key-positions.md)

## Building the Firmware with GitHub Actions

### Setup

1. Fork this repo.
2. Enable GitHub Actions on your fork.

### Build firmware

1. Push a commit to trigger the build.
2. Download the artifact.

## Building the Firmware in a local container (For advanced users only)

### Setup

#### Software

* Either Podman or Docker is required, Podman is chosen if both are installed.
* Make is also required

#### Windows specific

* If compiling on Windows use WSL2 and Docker [Docker Setup Guide](https://docs.docker.com/desktop/windows/wsl/).
* Install make using `sudo apt-get install make` inside the WSL2 instance.
* The repository can be cloned directly into the WSL2 instance or accessed through the C: mount point WSL provides by default (`/mnt/c/path-to-repo`).

### Build firmware

1. Execute `make`.
2. Check the `firmware` directory for the latest firmware build.

### Cleanup

The built docker container and compiled firmware files can be deleted with `make clean`. This might be necessary if you updated your fork from V2.0 to V3.0 and are encountering build failures. 

## Flashing firmware

Follow the firmware update instruction guide available [here](https://kinesis-ergo.com/support/kb360pro/#manual)

### Overview

1. Extract the firmwares from the archive downloaded from the GitHub build job (If using the cloud builder) or the firmware folder (If building locally).
1. Connect the left side keyboard to USB.
1. Press the physical reset button on the left side twice in rapid succession with a paperclip to put it into bootloader mode to attach it as a USB drive.
1. Copy `left.uf2` to the USB drive and it will disconnect.
1. Power off both keyboards (by unplugging them and making sure the switches are off).
1. Connect the right side keyboard to USB to power it on.
1. Press the physical reset button on the right side twice in rapid succession with a paperclip to put it into bootloader mode to attach it as a USB drive.
1. Copy `right.uf2` to the mounted drive.
1. Unplug the right side keyboard and turn both sides back on.
1. Enjoy!

> Note: The location of the physical reset buttons is described in section 2.2 and section 2.7 of the Advantage 360 Pro user manual (Available [here](https://kinesis-ergo.com/support/kb360pro/#manual)). There is also a key combination (Mod + HK1 for the left side and Mod + HK3 for the right side) however this *only works if the halves are connected to each other*.

> Note: Some operating systems wont always treat the drive as ejected after the settings-reset file is flashed or will give an error saying the copy process has failed, this doesn't mean that the flashing process has failed. For more information please see [the ZMK documentation](https://zmk.dev/docs/troubleshooting#file-transfer-error) on file transfer errors.

### Upgrading from V2 to V3

If you encounter a git conflict when updating your repository to V3.0 please follow the instructions on how to resolve it [here](UPGRADE.md).

Updating from V2.0 based firmwares to V3.0 based firmwares can be a rather complex process. There are reset files for every major firmware revision as well as documentation on the update process available [here](https://kinesis-ergo.com/support/kb360pro/#firmware-updates).

## Versioning

Starting on 11/15/2023 the Advantage 360 Pro will now automatically record the compilation date, branch and Git commit hash in a macro that can be accessed with Mod+V. This will type out the following string: YYYYMMDD-XXXX-YYYYYY, where XXXX is the first 4 characters of the Git branch and YYYYYY is the Git commit hash. In addition to this the builds compiled by GitHub actions are now timestamped and also record the commit hash in the filename. 

## N-Key Rollover

By default this keyboard has NKRO enabled, however for compatibility reasons the higher ranges are not enabled. If you want to use F13-F24 or the INTL1-9 keys with NKRO enabled you can change `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=n` to `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=y` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig#L65)

## Battery reporting

By default reporting the battery level over BLE is disabled as this can cause some computers to spontaneously wake up repeatedly. If you'd like to enable this functionality change `CONFIG_BT_BAS=n` to  `CONFIG_BT_BAS=y` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig#L58). Please note that a known bug in windows prevents the battery level from updating on older versions of windows, it is only updated when the board is paired. A workaround is to set `CONFIG_BT_GATT_ENFORCE_SUBSCRIPTION=n` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig). This may cause unexpected results on other OSes

## Changelog

The changelog for both the config repo and the underlying ZMK fork that the config repo builds against can be found [here](CHANGELOG.md).

## Troubleshooting

If you're having issues with your Advantage 360 Pro there are several things you can try. The most common issues we see users encountering are Bluetooth connection issues to the host PC. You can try unpairing from the device (Mod + win with the factory keymap) and unpairing on the host and connecting again, or check the [issues](https://github.com/KinesisCorporation/Adv360-Pro-ZMK/issues) to see if other users are experiencing the same problem as you and if there are any solutions

If you're having problems getting your firmware to compile (A red X in the GitHub build action page or emails about build failures) you can check some useful advice [here](https://zmk.dev/docs/troubleshooting#west-build-errors)

There are other troubleshooting resources hosted here: https://kinesis-ergo.com/support/kb360pro/#troubleshooting

The latest factory firmware can be found heere if you'd like to restore your keyboard to factory settings: https://kinesis-ergo.com/support/kb360pro/#firmware-updates

## Beta testing

The Advantage 360 Pro is always getting updates and refinements. If you are willing to beta test you can follow [this guide from ZMK](https://zmk.dev/docs/features/beta-testing#testing-features) on how to change where your config repo points to. The `west.yml` file that is mentioned is located in config/. [This link](config/west.yml) can take you to the file. Typically you will only need to change the `revision: ` to match the beta branch. The current beta branch is [adv360-z3.2-beta](https://github.com/ReFil/zmk/tree/adv360-z3.2-beta) which is compatible with V3.0 config repositories however users must [disable BT privacy](#bluetooth-le-privacy).

Feedback on beta branches should be submitted as a GitHub issue on the base ZMK repository as opposed to this config repository

In the event of a major update the beta branch may not be compatible with the current mainline version of the config repository. If this is the case it will be detailed here along with instructions on how to update.

## Note

By default this config repository references [a customised version of ZMK](https://github.com/ReFil/zmk/tree/adv360-z3.2) with Advantage 360 Pro specific functionality and changes over [base ZMK](https://github.com/zmkfirmware/zmk). The Kinesis fork is regularly updated to bring the latest updates and changes from base ZMK however will not always be completely up to date, some features such as new keycodes will not be immediately available on the 360 Pro after they are implemented in base ZMK.

Whilst the Advantage 360 Pro is compatible with base ZMK (The pull request to merge it can be seen [here](https://github.com/zmkfirmware/zmk/pull/1454) if you want to see how to implement it) some of the more advanced features (the indicator RGB leds) will not work, and Kinesis cannot provide customer service for usage of base ZMK. Likewise the ZMK community cannot provide support for either the Kinesis keymap editor, nor any usage of the Kinesis custom fork.

## Escalated support

Further support resources can be found on Kinesis.com:

* https://kinesis-ergo.com/support/kb360pro/#firmware-updates
* https://kinesis-ergo.com/support/kb360pro/#manuals

If you're encountering a specific reproducible issue with the hardware or software you can open a GitHub ticket.
* https://github.com/KinesisCorporation/Adv360-Pro-ZMK/issues/new

In the event of a hardware issue it may be necessary to open a support ticket directly with Kinesis as opposed to a GitHub issue in this repository.
* https://kinesis-ergo.com/support/contact-a-technician/