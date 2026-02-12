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
| xDot-ES | MAX32670 | `XDOT_MAX32670` | xDot Essentials - saves config/session to MCU flash |
| xDot-AD | MAX32670 | `XDOT_MAX32670` | xDot Advanced - EEPROM for config/session, external flash for FUOTA |

The xDot-ES and xDot-AD are based on the Analog Devices MAX32670 microcontroller.  Both share the same Mbed target and bootloader.

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

**xDot / xDot-ES:** The firmware image is written directly into MCU flash.

**xDot-AD:** The bootloader first moves the firmware image to external flash.  On reset, the bootloader validates the image from flash before applying it.  This results in additional time for the upgrade due to loading into and out of external flash.  A CRC is used to validate the image stored in flash.

> **Important:** Always use application-only firmware images for serial upgrades (files from the `APPS/` directory in [Dot-AT-Firmware](https://github.com/MultiTechSystems/Dot-AT-Firmware)).  Full images containing a bootloader will fail or produce unexpected results.  If you must use a full image, the upgrade tool will detect it and strip the bootloader automatically.

### xDot-ES vs xDot-AD Trade-offs

xDot-ES uses MCU flash for config/session storage instead of EEPROM.  Trade-offs of MCU flash include limited write endurance compared to EEPROM and contention between MCU flash operations and application execution.


## Enabling External Storage on xDot

xDot requires an external storage device for FOTA.  Space is allocated to store the received file, a backup copy of the application, and an upgrade result file.

> **Note:** This section applies to the original xDot (`XDOT_L151CC`) and xDot-ES, which do not have external flash.  xDot-AD has external flash for FUOTA and does not require additional external flash configuration.

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
