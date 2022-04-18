# XDot DAPLink Interface

The XDot development kit has a DAPLink interface firmware which enables developers with drag-and-drop programming, a serial port, and CMSIS-DAP based debugging.

Get the latest XDot DAPLink interface firmware [here](https://github.com/MultiTechSystems/DAPLink/releases/latest)

*Note: DAPLink release v0254 does no work with XDot.*


## Programming the interface chip

Steps for programming the Kinetis MK20DX128 DAPLink interface chip:

1. Hold RESET on XDot DK while attaching USB
2. Copy new interface firmware file to the MAINTENANCE drive
3. Open DETAILS.TXT on the XDot drive
4. Verify `Interface Version` is the new version


## Configure for auto-reset

By default the XDot will require a manual reset after programming.  With auto-reset enabled the XDot will reset after programming automatically.

Steps for enabling auto-reset:

1. Create an empty file names `auto_rst.cfg`
2. Hold RESET on the XDot DK
3. Drop `auto_rst.cfg` into the XDot drive
4. Release RESET
5. Open DETAILS.TXT on the XDot drive
6. Verify `Auto Reset: 1`

For more information on DAPLink configuration options see [MSD_COMMANDS](https://github.com/MultiTechSystems/DAPLink/blob/master/docs/MSD_COMMANDS.md).


## Using with pyOCD

[PyOCD](https://pypi.org/project/pyocd/) can be used for XDot programming and debugging.

Use this command to flash new firmware to an XDot:

```
pyocd flash -M under-reset -e sector <xdot-firmware>
```
