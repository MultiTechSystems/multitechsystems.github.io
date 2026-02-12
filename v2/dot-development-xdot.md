---
layout: default
title: "Dot Development - xDot, xDot-ES & xDot-AD"
permalink: /v2/dot-development-xdot/
doc_version: v2
---

# Dot Development - xDot Family

This article covers topics specific to the xDot family of modules.

| Module  | MCU | Mbed Target | Description |
|---------|-----|-------------|-------------|
| xDot    | STM32L151CC | `XDOT_L151CC` | Original xDot module |
| xDot-ES | MAX32670 | `XDOT_MAX32670` | xDot with enhanced security |
| xDot-AD | MAX32670 | `XDOT_MAX32670` | xDot with enhanced security and additional debug interface |

The xDot-ES and xDot-AD are based on the Analog Devices MAX32670 microcontroller.  xDot-AD adds an additional debug serial interface.  Both share the same Mbed target and bootloader.

See also: [Dot Development](/v2/dot-development/) for general topics applicable to all Dot modules.


## Serial Firmware Upgrade

The recommended way to update firmware on xDot family devices over serial is the [Dot Upgrade Tool](https://github.com/MultiTechSystems/Dot-AT-Firmware) included in the Dot-AT-Firmware repository.  It handles the differences between device types automatically.

```
python tools/dot-upgrade.py upgrade XDOT US915 /dev/ttyUSB0
python tools/dot-upgrade.py upgrade XDOTES EU868 COM3
python tools/dot-upgrade.py upgrade XDOTAD AS923 /dev/ttyACM0
```

See the [Programming](/v2/dot-development/#programming) section in Dot Development for full setup instructions.

### Device-Specific Notes

**xDot** uses the `upgrade` bootloader command and does **not** require a CRC.  Application-only images can be sent directly.

**xDot-ES** uses the `upgrade` bootloader command and does **not** require a CRC.  Application-only images can be sent directly.

**xDot-AD** uses the `upgrade` bootloader command and **requires a CRC32** appended to the image.  The upgrade tool handles this automatically.  After transfer, the device verifies the CRC before applying the upgrade.  The complete upgrade process takes approximately 90 seconds.  Do not power off or reset the device during this time.

> **Important:** Always use application-only firmware images for serial upgrades (files from the `APPS/` directory in [Dot-AT-Firmware](https://github.com/MultiTechSystems/Dot-AT-Firmware)).  Full images containing a bootloader will fail or produce unexpected results.  If you must use a full image, the upgrade tool will detect it and strip the bootloader automatically.


### Bootloader Upgrade Flow

The serial upgrade sequence for xDot family devices:

1. Device is in AT command mode (or bootloader mode)
2. Tool sends `ATZ` to reset, then `mts` to enter the bootloader
3. Bootloader prompt `bootloader :>` appears
4. Tool sends the `upgrade` command
5. Device signals ready with YMODEM `C` character
6. Tool transfers the prepared firmware file via YMODEM
7. **xDot / xDot-ES:** Upgrade is applied immediately after transfer
8. **xDot-AD:** Device verifies CRC, then applies upgrade (~90 seconds)

### xDot-AD CRC Verification

If the CRC check fails on xDot-AD, the bootloader will print an `ERROR` message and the device will not boot the new firmware.  The bootloader will remain active.  In this case:

* Verify the correct application image was used
* Ensure the CRC was appended correctly (the upgrade tool does this automatically)
* Retry the upgrade


## Enabling External Storage on xDot

xDot requires an external storage device for FOTA.  Space is allocated to store the received file, a backup copy of the application, and an upgrade result file.

> **Note:** This section applies to the original xDot (`XDOT_L151CC`).  xDot-ES and xDot-AD have built-in storage and do not require external flash configuration.

Storage devices must meet the following criteria:

* Work with Mbed OS DataFlashBlockDevice or SPIFBlockDevice classes
* Maximum 4KB sector erase size
* Maximum 512 byte page size
* SPIF type components must support Serial Flash Discoverable Parameters (SFDP)
* 512KB of free space


These parts have been tested:

* Macronix MX25R8035
* Adesto AT45DB041E


The block device driver must be configured in `mbed_app.json`.

Example SPIF Block device configuration:

```json
{
    ...

    "target_overrides": {
        ...

        "XDOT_L151CC": {
            ...

            "target.components_add": ["SPIF"],

            "spif-driver.SPI_MOSI": "SPI_MOSI",
            "spif-driver.SPI_MISO": "SPI_MISO",
            "spif-driver.SPI_CLK": "SPI_SCK",
            "spif-driver.SPI_CS": "SPI_NSS",

            "flash-record-store.file-enable": 1,
            "flash-record-store.ext-flash-page-size": 512,
            "flash-record-store.ext-flash-sector-size": 4096,
            "flash-record-store.ext-flash-erase-size": 4096
        }
    }
}
```

Example DATAFLASH block device configuration:

```json
{
    ...

    "target_overrides": {
        ...

        "XDOT_L151CC": {
            ...

            "target.components_add": ["DATAFLASH"],

            "dataflash.SPI_MOSI": "SPI_MOSI",
            "dataflash.SPI_MISO": "SPI_MISO",
            "dataflash.SPI_CLK": "SPI_SCK",
            "dataflash.SPI_CS": "SPI_NSS",
            "dataflash.binary-size": 1,

            "flash-record-store.file-enable": 1,
            "flash-record-store.ext-flash-page-size": 512,
            "flash-record-store.ext-flash-sector-size": 4096,
            "flash-record-store.ext-flash-erase-size": 4096
        }
    }
}
```


The mDot instance must be initialized with a block device instance.  The xDot bootloader expects file storage to be allocated at the end of the flash device.

Example for initializing an SPIF block device:

```cpp
#include "mbed.h"
#include "mDot.h"

#include "SPIFBlockDevice.h"
// #include "DataFlashBlockDevice.h"

...


int main()
{
    ...

    mbed::BlockDevice* ext_bd = NULL;

    ext_bd = new SPIFBlockDevice();
    // ext_bd = new DataFlashBlockDevice();
    int ret = ext_bd->init();
    // Check for zero size because DataFlashBlockDevice doesn't
    // return an error if the chip is not present
    if (ret < 0 || (ext_bd->size() == 0)) {
        delete ext_bd;
        ext_bd = NULL;
    }

    mDot* dot = mDot::getInstance(&plan, ext_bd);

    if (ext_bd != NULL) {
        logInfo("External flash device detected, type: %s, size: 0x%08x",
            ext_bd->get_type(), (uint32_t)ext_bd->size());
    }

    ...
}
```
