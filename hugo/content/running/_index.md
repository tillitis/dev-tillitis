---
title: Running
weight: 3
---

# Running

How do you run the host and device programs?

Build our [qemu](https://github.com/tillitis/qemu). Use the `tk1`
branch. Please follow the
[toolchain_setup.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/toolchain_setup.md)
and install the packages listed there first.


```
$ git clone -b tk1 https://github.com/tillitis/qemu
$ mkdir qemu/build
$ cd qemu/build
$ ../configure --target-list=riscv32-softmmu --disable-werror
$ make -j $(nproc)
```

(Built with warnings-as-errors disabled, see [this
issue](https://github.com/tillitis/qemu/issues/3).)

You also need to build the firmware:

```
$ git clone https://github.com/tillitis/tillitis-key1
$ cd tillitis-key1/hw/application_fpga
$ make firmware.elf
```

Please refer to the mentioned
[toolchain_setup.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/toolchain_setup.md)
if you have any issues building.

Then run the emulator, passing using the built firmware to "-bios":

```
$ /path/to/qemu/build/qemu-system-riscv32 -nographic -M tk1,fifo=chrid -bios firmware.elf \
  -chardev pty,id=chrid
```

It tells you what serial port it is using, for instance `/dev/pts/1`.
This is what you need to use as `--port` when running the host
programs.

The TK1 machine running on QEMU (which in turn runs the firmware, and
then the app) can output some memory access (and other) logging. You
can add `-d guest_errors` to the qemu commandline To make QEMU send
these to stderr.

Plug the USB stick into your computer. If the LED in one of the outer
corners of the USB stick is flashing white, then it has been
programmed with the standard FPGA bitstream (including the firmware).
If it is not then please refer to
[quickstart.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/quickstart.md)
(in the tillitis-key1 repository) for instructions on initial
programming of the USB stick.

## Users on Linux

Running `lsusb` should list the USB stick as `1207:8887 Tillitis
MTA1-USB-V1`. On Linux, the TKey's serial port device path is
typically `/dev/ttyACM0` (but it may end with another digit, if you
have other devices plugged in). The host programs tries to auto-detect
serial ports of TKey USB sticks, but if more than one is found you'll
need to choose one using the `--port` flag.

However, you should make sure that you can read and write to the
serial port as your regular user.

One way to accomplish this is by installing the provided
`system/60-tkey.rules` in `/etc/udev/rules.d/` and running `udevadm
control --reload`. Now when a TKey is plugged in, its device path
(like `/dev/ttyACM0`) should be read/writable by you who are logged in
locally (see `loginctl`).

Another way is becoming a member of the group that owns the serial
port. On Ubuntu that group is `dialout`, and you can do it like this:

```
$ id -un
exampleuser
$ ls -l /dev/ttyACM0
crw-rw---- 1 root dialout 166, 0 Sep 16 08:20 /dev/ttyACM0
$ sudo usermod -a -G dialout exampleuser
```

For the change to take effect everywhere you need to logout from your
system, and then log back in again. Then logout from your system and
log back in again. You can also (following the above example) run
`newgrp dialout` in the terminal that you're working in.

Your TKey is now running the firmware. Its LED flashing white,
indicating that it is ready to receive an app to run.

## User on MacOS

You can check that the OS has found and enumerated the USB stick by
running:

```
ioreg -p IOUSB -w0 -l
```

There should be an entry with `"USB Vendor Name" = "Tillitis"`.

Looking in the `/dev` directory, there should be a device named like
`/dev/tty.usbmodemXYZ`. Where XYZ is a number, for example 101. This
is the device path that might need to be passed as `--port` when
running the host programs.
