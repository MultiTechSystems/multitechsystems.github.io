# Dot Development

This article provides general project configuration and programming for working with Dots.

Related articles:

* [Migrating to 4.0](dot-migrate-to-4) - Major changes to Dot libraries between 3.3 and 4.0

* [xDot](dot-development-xdot) - xDot specific topics

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

The Mbed online compiler outputs only a binary that is a merged bootloader and application.  Compiling offline with Mbed CLI or Mbed Studio will output a merged binary and an application only binary.  A full merged image (bootlaoder and application) is used when programming a Dot with a programmer or drag-and-drop.  Updating devices over serial or FOTA requires an application-only binary.

[Multitool](https://pypi.org/project/mtsmultitool) can be used to strip the bootloader from a binary and program a device over serial.


Example multitool commands for stripping the bootloader and appending a CRC:
```
multitool device plain -b -i MTDOT -c -o mdot_app.bin mbed_output.bin
multitool device plain -b -i XDOT -c -o xdot_app.bin mbed_output.bin
```

Example multitool commands for appending a CRC only:
```
multitool device plain -c -o mdot_app.bin mbed_output_application.bin
multitool device plain -c -o xdot_app.bin mbed_output_application.bin
```

Example multitool commands for stripping the bootloader, appending a CRC, and sending an upgrade to a device over serial.
```
multitool device upgrade -b COM3 MTDOT mbed_output.bin
multitool device upgrade -b COM3 XDOT mbed_output.bin
```
