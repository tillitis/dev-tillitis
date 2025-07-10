---
title: USB Controller Firmware
weight: 5
---

# USB Controller Firmware

The USB controller on the TKey is a WCH CH552 with its own firmare.

The TKey, even the TKey Unlocked, usually comes with the USB
controller firmware already flashed. If you need to change it for some
reason, perhaps if you have an older TKey Unlocked and changes have
been done to the CH552 firmware, you need to flash it yourself.

The firmware is kept in
[usb_interface/ch552_fw](https://github.com/tillitis/tillitis-key1/tree/main/hw/usb_interface/ch552_fw).
To build it you need `sdcc`. It's included in the
[tkey-builder](https://ghcr.io/tillitis/tkey-builder) OCI image.

You also need [chprog](https://github.com/tillitis/chprog/) to actually
flash the firmware.

You need a CH55x Reset Controller. You can buy it from
[Blinkinlabs](https://shop-nl.blinkinlabs.com/products/ch55x-reset-controller)
or build it yourself. You need to perform this sequence:

1. Disconnect both power and USB data lines from the device.
2. Connect a 10k resistor from 3.3V to the D+ USB line.
3. Connect the power and USB data lines to the device.
4. After a short delay, disconnect the 10k resistor from the device.

To be able to use chprog without having to be root you have to install:

```
# CH552 bootloader (for unprogrammed CH552 chips)
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTR{idVendor}=="4348", ATTR{idProduct}=="55e0", MODE="0666", GROUP="dialout"
```

in `/etc/udev/rules.d` and do `udevadm control --reload`.

To use the Blinkinlabs Reset Controller:

1. Connect your computer to `DUT_IN`.
2. Connect the TKey to be flashed to `DUT_OUT`
3. Press the button marked "bootloader". Note that there are two
   buttons.
4. Run `make flash_patched` in `hw/usb_interface/ch552_fw` which runs
   chprog.
