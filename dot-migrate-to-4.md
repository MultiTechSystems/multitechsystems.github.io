# Dot Development - Migrating to 4.0

Migrating an existing project from Dot libraries v3.3 to v4.0 will require some changes to compile and function properly.  The project must use mbed-os 6, the 4.0 libraries are compiled against mbed-os 6.1.0 and are only tested with that version of mbed-os.

This article describes some major changes to the Dot libraries and bootloader in the 4.0 release version that effect project setup and programming.  It does not cover changes to LoRa.

## Major Changes

* The `Serial` class was replaced by `UnbufferedSerial` and `BufferedSerial` in mbed-os 6.

* Device settings on mDot have been migrated from the file system to a separate storage area.  

* External storage support on xDot enabling FOTA.

* Applications using the 4.0 libraries and mbed-os 6 are not compatible with the bootloader that was automatically included by mbed-os 5.

* Projects targeting mDot or xDot can now choose to include or exclude a bootloader.

* New bootloader supports compression and binary difference firmware upgrades.

* When applying an upgrade the mDot bootloader no longer assumes the file contains a bootloader.

* A CRC it no longer automatically appended to firmware when compiled with Mbed.


## Settings Migration

Settings stored on mDot are migrated automatically when `mDot::getInstance` is first called.  If the new settings area is not found, the mDot loads settings from files `lora.cfg`, `mdot.cfg`, and `lora.session` then erases the entire storage area.  Storage is then allocated with 4 sectors allocated to settings and 28 allocated to the file system (a sector is 64KB).


## Serial 

The `Serial` class was replaced by `UnbufferedSerial` and `BufferedSerial` classes in mbed-os 6.  Adding the example code below creates an `UnbufferedSerial` instance and sets it as the default printf `FileHandle`.

```cpp
#include "mbed.h"

...

mbed::UnbufferedSerial debug_port(USBTX, USBRX, LOG_DEFAULT_BAUD_RATE);

FileHandle *mbed::mbed_override_console(int fd)
{
    return &debug_port;
}

...
```


## Bootloader

* A CRC it no longer automatically appended to firmware when compiled with Mbed.

When upgrading a dot over serial the bootloader uses the appended CRC to verify the received file.  Sending a file to a Dot over serial will require calculating and appending a CRC32 to the file before transfering.  [Multitool](https://pypi.org/project/mtsmultitool) can be used to append a CRC and transfer the file.


* When applying an upgrade the mDot bootloader no longer assumes the file contains a bootloader.

Firmware images sent to the new bootloader must be application only.  The old bootloader requires upgrade images to be a merged bootloader and application.  [Multitool](https://pypi.org/project/mtsmultitool) can be used to strip the bootloader from a file.

