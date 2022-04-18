# Dot Development - xDot


## Enabling external storage on XDot

XDot requires an external storage device for FOTA.  Space is allocated to store the received file, a backup copy of the application, and an upgrade result file.

Storage devices must meet the following criteria:

* Work with MBed OS DataFlashBlockDevice or SPIFBlockDevice classes
* Maximum 4KB sector erase size
* Maximum 512 byte page size
* SPIF type components must support Serial Flash Discoverable Parameters (SFDP)
* 512KB of free space. 


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
