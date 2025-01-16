---
title: USB Controller Firmware
weight: 5
---

# USB Controller Firmware

The USB controller on the TKey is a WCH CH552 with its own firmare.

The TKey, even the TKey Unlocked, usually comes with the USB
controller firmware already flashed. If you need to change it for some
reason, perhaps if you have an older Unlocked and changes has been
done to the CH552 firmware, you need to flash it yourself.

The firmware is kept in
[usb_interface/ch552_fw](https://github.com/tillitis/tillitis-key1/tree/main/hw/usb_interface/ch552_fw).
To build it you need `sdcc`. It's included in the `tkey-builder`
image.

You need a CH55x Reset Controller. You can buy it from
[Blinkinlabs](https://shop-nl.blinkinlabs.com/products/ch55x-reset-controller)
or build it yourself. You need to perform this sequence:

1. Disconnect both power and USB data lines from the device.
2. Connect a 10k resistor from 3.3V to the D+ USB line.
3. Connect the power and USB data lines to the device.
4. After a short delay, disconnect the 10k resistor from the device.

To use the Blinkinlabs one:

1. Connect your computer to `DUT_IN`.
2. Connect the TKey to be flashed to `DUT_OUT`
3. Press the button marked "bootloader". Note that there are two
   buttons.

This will make the CH552 enter its internal bootloader. When it does,
use [chprog](https://github.com/ole00/chprog/) to feed it with the
firmware.

Use `make flash_patched` in the controller source to actually write it
to the CH552's waiting bootloader.

TODO: Do they need to do something with `inject_serial_number.py`?
