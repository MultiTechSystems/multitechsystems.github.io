---
layout: default
title: Dot Development
description: Project configuration and programming guide for MultiTech Dot modules
category: development
tags: [mdot, xdot, xdot-es, xdot-ad, embedded, lora]
permalink: /v2/dot-development/
doc_version: v2
---

# Dot Development

This article provides general project configuration and programming for working with Dots.

Related articles:

* [Migrating to 4.0](/v2/dot-migrate-to-4/) - Major changes to Dot libraries between 3.3 and 4.0

* [xDot / xDot-ES / xDot-AD](/v2/dot-development-xdot/) - xDot specific topics including external storage configuration


## Supported Devices

| Device  | MCU | Target | Application Offset |
|---------|-----|--------|--------------------|
| mDot    | STM32F411RE | `MTS_MDOT_F411RE` | `0x10000` |
| xDot    | STM32L151CC | `XDOT_L151CC` | `0xD000` |
| xDot-ES | MAX32670 | `XDOT_MAX32670` | `0x10000` |
| xDot-AD | MAX32670 | `XDOT_MAX32670` | `0x10000` |


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

A bootloader must be specified in `mbed_app.json` along with the application offset. Bootloader binaries can be obtained from [https://github.com/MultiTechSystems/bootloaders](https://github.com/MultiTechSystems/bootloaders).

Define `app_offset` and `bootloader_img` target overrides in `mbed_app.json`.  These settings tell Mbed how to merge the bootloader and application images.

Example configuration for mDot:
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

Example configuration for xDot-ES / xDot-AD:
```json
{
    ...

    "target_overrides": {
        ...

        "XDOT_MAX32670": {
            "target.app_offset": "0x10000",
            "target.bootloader_img": "bootloaders/xdot-max32670-bootloader.bin"
        }
    }
}
```

Note `target.app_offset` is ignored if `target.bootloader_img` is not present.


## Programming

A full merged image (bootloader and application) is used when programming a Dot with a programmer or drag-and-drop.  **Updating devices over serial or FOTA requires an application-only binary.**

> **Important:** Do not send a full image (bootloader + application) through the serial bootloader.  The bootloader must be stripped first or an application-only binary must be used.  Some devices also require a CRC32 to be appended.  The upgrade tool handles this automatically.


### Serial Firmware Upgrade Tool (Recommended)

The [Dot-AT-Firmware](https://github.com/MultiTechSystems/Dot-AT-Firmware) repository includes a cross-platform Python upgrade tool that automates serial firmware updates.  It replaces the TeraTerm dependency and handles image preparation (bootloader stripping and CRC) automatically.

**Setup:**
```
git clone https://github.com/MultiTechSystems/Dot-AT-Firmware.git
cd Dot-AT-Firmware
pip install -r tools/requirements.txt
```

**Upgrade using firmware from the repository:**
```
python tools/dot-upgrade.py upgrade MDOT US915 /dev/ttyUSB0
python tools/dot-upgrade.py upgrade XDOT EU868 COM3
python tools/dot-upgrade.py upgrade XDOTES US915 /dev/ttyUSB0
python tools/dot-upgrade.py upgrade XDOTAD AS923 COM5
```

**Upgrade with a custom firmware file:**
```
python tools/dot-upgrade.py upgrade MDOT US915 /dev/ttyUSB0 --file my_app.bin
```

If a full image is provided, the tool will automatically detect it, strip the bootloader, and append a CRC if the device requires one.

**Prepare an image without sending (offline processing):**
```
python tools/dot-upgrade.py prepare MDOT full_image.bin -o upgrade_ready.bin
python tools/dot-upgrade.py prepare XDOTAD application.bin -o upgrade_ready.bin
```

**Launch the graphical interface:**
```
python tools/dot-upgrade.py --gui
```

**Other useful commands:**
```
python tools/dot-upgrade.py list                  # List available firmware
python tools/dot-upgrade.py list --device XDOTAD  # Filter by device
python tools/dot-upgrade.py ports                 # List serial ports
python tools/dot-upgrade.py versions              # List firmware versions
```

The tool automatically handles the device-specific differences:

| Device  | CRC Required | Bootloader Command | Upgrade Wait |
|---------|--------------|--------------------|--------------|
| mDot    | Yes (auto)   | `upgrade ymodem`   | ~30 seconds  |
| xDot    | No           | `upgrade`          | Immediate    |
| xDot-ES | No           | `upgrade`          | Immediate    |
| xDot-AD | Yes (auto)   | `upgrade`          | ~90 seconds  |


### Bootloader Entry

To enter the bootloader for serial firmware upgrades, the device must be reset and a key sequence must be received within 250ms of boot.  The upgrade tool handles this automatically, but the behavior varies by bootloader version and serial interface.

**Bootloader key by version:**

| Bootloader Version | AT/Command Port | Debug Port | Notes |
|--------------------|----------------|------------|-------|
| v0.1.x             | N/A            | Any keypress | Single port only (xDot mbed-os-5) |
| 1.0.0 - 1.1.9      | `mts`          | Any keypress | STM32 and MAX32670 targets |
| 1.2.0+             | `mts`          | `mts` (MAX32670) / Any keypress (others) | MAX32670 debug port now requires key |

> The key `xdt` was used in some intermediate xDot bootloader builds but was changed to `mts` before any tagged release.  The upgrade tool tries both keys for compatibility.

**How the upgrade tool enters the bootloader:**

* **Debug port** (`--debug-port` or auto-detected): Sends a serial break to reset the device, watches for the `[INFO] MultiTech Bootloader x.x.x` banner on the debug output, parses the version number, and selects the correct key automatically.  For versions < 1.2.0, any keypress is sufficient.  For version >= 1.2.0 on MAX32670 targets (xDot-ES/AD), the tool sends `mts`.  Falls back to prompting for manual reset if break does not work.
* **AT command port** (`--at-port` or auto-detected): The bootloader version is not visible on the AT port, so the tool sends `ATZ` to reset, then tries `mts` (then `xdt` as fallback) via echo-detection (the bootloader echoes each character, confirming it is running).

The bootloader version is displayed in the debug output banner during boot (e.g., `[INFO] MultiTech Bootloader 1.1.7`).  The bootloader source is available at [gitlab.multitech.net/mbed/generic-bootloader](https://gitlab.multitech.net/mbed/generic-bootloader).


### Multitool (Legacy)

[Multitool](https://pypi.org/project/mtsmultitool) is a pip-installable package that can strip the bootloader from a binary, append a CRC, create BSDIFF patches, compress images with LZ4, and generate SUIT manifests for FOTA. It supports mDot and xDot.

Example commands for stripping the bootloader and appending a CRC:
```
multitool device plain -b -i MTDOT -c -o mdot_app.bin mbed_output.bin
multitool device plain -b -i XDOT -c -o xdot_app.bin mbed_output.bin
```

Example commands for appending a CRC only:
```
multitool device plain -c -o mdot_app.bin mbed_output_application.bin
multitool device plain -c -o xdot_app.bin mbed_output_application.bin
```

Example commands for serial upgrade:
```
multitool device upgrade -b COM3 MTDOT mbed_output.bin
multitool device upgrade -b COM3 XDOT mbed_output.bin
```
