---
title: Tools
weight: 2
---

# Tools

## Toolchain

To build you need at least `clang`, `llvm`, `lld`, `golang` packages
installed. Version 15 or later of LLVM/Clang is required (with riscv32
support and the Zmmul extension, `-march=rv32iczmmul`). Packages on
Ubuntu 22.10 (Kinetic) are known to work.

On Ubuntu you can do:

```
sudo apt install build-essential clang lld llvm bison flex libreadline-dev \
                 gawk tcl-dev libffi-dev git mercurial graphviz \
                 xdot pkg-config python3 libftdi-dev \
                 python3-dev libeigen3-dev \
                 libboost-dev libboost-filesystem-dev \
                 libboost-thread-dev libboost-program-options-dev \
                 libboost-iostreams-dev cmake libhidapi-dev \
                 ninja-build libglib2.0-dev libpixman-1-dev \
                 golang clang-format
```

## Toolchain container

As a convenience we provide a Docker/Podman image which has all these
tools already installed

```
$ podman pull ghcr.io/tillitis/tkey-builder:latest
```

TODO Add example of running.

## QEMU

We have a TKey emulator based on QEMU. 

Build our [qemu](https://github.com/tillitis/qemu). Use the `tk1`
branch.

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

Then run the emulator, passing using the built firmware to "-bios":

```
$ /path/to/qemu/build/qemu-system-riscv32 -nographic -M tk1,fifo=chrid -bios firmware.elf \
  -chardev pty,id=chrid
```

It tells you what serial port it is using, for instance `/dev/pts/1`.
This is what you need to use as `--port` when running the host
programs.

The TK1 machine running on QEMU (which in turn runs the firmware, and
then the TKey program) can output some memory access (and other)
logging. You can add `-d guest_errors` to the qemu command line to
make QEMU send these to stderr.

## Building our TKey programs

Clone our TKey program repo:

```
$ git clone https://github.com/tillitis/tillitis-key1-apps.git
$ cd tillitis-key1-apps
```

To build everything:

```
$ make
```

If your available `objcopy` is anything other than the default
`llvm-objcopy`, then define `OBJCOPY` to whatever they're called on
your system.

The TKey program can be run both on the hardware TKey, and on a QEMU
machine that emulates the platform. In both cases, the client program
(the program that runs on your computer, for example `tkey-ssh-agent`)
will talk to the TKey program over a serial port, virtual or real.
There is a separate section below which explains running in QEMU.

## Running TKey programs

Plug the USB stick into your computer. If the LED in one of the outer
corners of the USB stick is flashing white, then it has been
programmed with the standard FPGA bitstream (including the firmware).
If it is not then please refer to
[quickstart.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/quickstart.md)
(in the tillitis-key1 repository) for instructions on initial
programming of the USB stick.

### Users on Linux

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

#### User on MacOS

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

## Developing TKey program

TKey programs and libraries are kept under the `apps` directory. A C
runtime is provided as `apps/libcrt0/libcrt0.a` which you can link
your C programs with.

### Memory

RAM starts at 0x4000\_0000 and ends at 0x4002\_0000. The app will be
loaded by firmware at 0x4000\_0000. The assembler program in
`apps/libcrt0/crt0.S` sets up a 28 kiB stack at the top of RAM.

There are no heap allocation functions, no `malloc()` and friends. You
can access memory directly yourself. We provide `APP_ADDR` and
`APP_SIZE` so the loaded TKey program knows where it's loaded and how
large it is.

Special memory areas for memory mapped hardware functions are
available at base 0xc000\_0000 and an offset. See [the memory
map](../memory/). and the include file `tk1_mem.h`.

### Debugging

If you're running the app on our qemu emulator we have added a debug
port on 0xfe00\_1000 (`TK1_MMIO_QEMU_DEBUG`). Anything written there
will be printed as a character by qemu on the console.

`qemu_putchar()`, `qemu_puts()`, `qemu_putinthex()`, `qemu_hexdump()`
and friends (see `apps/libcommon/lib.[ch]`) use this debug port to
print stuff.

`libcommon` is compiled with no debug output by default. Rebuild
`libcommon` without `-DNODEBUG` to get the debug output.
