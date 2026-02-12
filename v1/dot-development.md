---
layout: default
title: "Dot Development"
permalink: /v1/dot-development/
doc_version: v1
---

# Dot Development

This article provides general project configuration and programming for working with Dots.

Related articles:

* [Migrating to 4.0](/v1/dot-migrate-to-4/) - Major changes to Dot libraries between 3.3 and 4.0

* [xDot](/v1/dot-development-xdot/) - xDot specific topics

## Application Configuration

Create mbed_app.json at the project root if one does not already exist.  The example below defines a baud rate for serial log output, enables FOTA, and sets the printf library.

```json
{
    "config": {
        "log-baud-rate" :  {
            "macro_name": "LOG_DEFAULT_BAUD_RATE",
            "value": 115200
        }
    },
    "macros": [
        "FOTA"
    ],
    "target_overrides": {
        "*": {
            "target.printf_lib": "std"
        }
    }
}
```


## Adding a Bootloader

A bootloader must be specified in `mbed_app.json` along with the application offset. Bootloader binaries can be obtained from https://github.com/MultiTechSystems/bootloaders.

Define `app_offset` and `bootloader_img` target overrides in `mbed_app.json`.  These settings tell mbed how to merge the bootloader and application images.

Targets `MTS_MDOT_F411RE`, `MTS_DRAGONFLY_F411RE`, and `MTS_DRAGONFLY_L471QG` have a `target.app_offset` of `0x10000`.  Target `XDOT_L151CC` has  `target.app_offset` of `0xD000`.

Example configuration for mDot, Dragonfly, and Dragonfly Nano:
```json
{
    ...

    "target_overrides": {
        ...

        "MTS_MDOT_F411RE": {
            "target.app_offset": "0x10000",
            "target.bootloader_img": "bootloaders/mdot-bootloader.bin"
        }
    }
}
```

Example configuration for xDot:
```json
{
    ...

    "target_overrides": {
        ...

        "XDOT_L151CC": {
            "target.app_offset": "0x0D000",
            "target.bootloader_img": "bootloaders/xdot-bootloader.bin"
        }
    }
}
```

Note `target.app_offset` is ignored if `target.bootloader_img` is not present.


## Programming

A full merged image (bootloader and application) is used when programming a Dot with a programmer or drag-and-drop.  **Updating devices over serial or FOTA requires an application-only binary.**

> **Important:** Do not send a full image (bootloader + application) through the serial bootloader.  The bootloader must be stripped first or an application-only binary must be used.


### Serial Firmware Upgrade Tool (Recommended)

The [Dot-AT-Firmware](https://github.com/MultiTechSystems/Dot-AT-Firmware) repository includes a cross-platform Python upgrade tool that automates serial firmware updates for mDot, xDot, xDot-ES, and xDot-AD.  It handles image preparation (bootloader stripping and CRC) automatically.

```
git clone https://github.com/MultiTechSystems/Dot-AT-Firmware.git
cd Dot-AT-Firmware
pip install -r tools/requirements.txt

python tools/dot-upgrade.py upgrade MDOT US915 COM3
python tools/dot-upgrade.py upgrade XDOTAD EU868 /dev/ttyUSB0
python tools/dot-upgrade.py --gui
```

See the [Dot-AT-Firmware README](https://github.com/MultiTechSystems/Dot-AT-Firmware) for full usage details.


### Bootloader Entry

To enter the bootloader for serial upgrades, the device must be reset and a key sequence must be received within 250ms of boot.  The upgrade tool handles this automatically.

| Bootloader Version | AT/Command Port | Debug Port |
|--------------------|----------------|------------|
| v0.1.x             | N/A            | Any keypress |
| 1.0.0 - 1.1.9      | `mts`          | Any keypress |
| 1.2.0+             | `mts`          | `mts` (MAX32670) / Any keypress (others) |

On the debug port, the tool sends a serial break to reset the device, parses the bootloader version from the `[INFO] MultiTech Bootloader x.x.x` banner, and selects the correct key automatically.  On the AT port, the bootloader version is not visible, so the tool sends `ATZ` to reset and tries `mts` then `xdt` via echo-detection.  Falls back to prompting for manual reset if needed.

The bootloader source is available at [gitlab.multitech.net/mbed/generic-bootloader](https://gitlab.multitech.net/mbed/generic-bootloader).


### Multitool (Legacy)

[Multitool](https://pypi.org/project/mtsmultitool) can be used to strip the bootloader from a binary and program a device over serial (mDot and xDot only).

Example multitool commands for stripping the bootloader and appending a CRC:
```
multitool device plain -b -i MTDOT -c -o mdot_app.bin mbed_output.bin
multitool device plain -b -i XDOT -c -o xdot_app.bin mbed_output.bin
```

Example multitool commands for serial upgrade:
```
multitool device upgrade -b COM3 MTDOT mbed_output.bin
multitool device upgrade -b COM3 XDOT mbed_output.bin
```
