---
title: Tools & libraries
weight: 2
---

# Tools & libraries

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
                 libboost-iostreams-dev cmake libusb-1.0-0-dev \
                 ninja-build libglib2.0-dev libpixman-1-dev \
                 golang clang-format
```

## Toolchain container

As a convenience we provide a container image which has all these
tools already installed, for use wit Podman or Docker.

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
This is what you need to use as `--port` when running the client
programs.

## Software Development Kit, or, Building our TKey programs

There not yet any stand-alone Software Development Kit. Instead, we
provide examples in the
[tillitis-key1-apps](https://github.com/tillitis/tillitis-key1-apps)
Github repo.

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

Plug the TKey into your computer. If the LED in one of the outer
corners of the TKey is white, then it has been programmed with the
standard FPGA bitstream (including the firmware). If it is not then
please refer to
[quickstart.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/quickstart.md)
(in the tillitis-key1 repository) for instructions on initial
programming of an unprovisioned TKey.

The examples below refer to files in the
[tillitis-key1-apps repository](https://github.com/tillitis/tillitis-key1-apps).

### Users on Linux

Running `lsusb` should list the USB stick as `1207:8887 Tillitis
MTA1-USB-V1`. On Linux, the TKey's serial port device path is
typically `/dev/ttyACM0` (but it may end with another digit, if you
have other devices plugged in). The client programs tries to
auto-detect TKeys, but if more than one is found you'll need to choose
one using the `--port` flag.

However, you should make sure that you can read and write to the
serial port as your regular user.

One way to accomplish this is by installing the provided
`system/60-tkey.rules` in `/etc/udev/rules.d/` and running `udevadm
control --reload`. Now when a TKey is plugged in its device path (like
`/dev/ttyACM0`) should be accessible by anyone logged in on the
console (see `loginctl`).

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
system and then log back in again. You can also run `newgrp dialout`
in the terminal that you're working in.

### Users on MacOS

You can check that the OS has found and enumerated the USB stick by
running:

```
ioreg -p IOUSB -w0 -l
```

There should be an entry with `"USB Vendor Name" = "Tillitis"`.

Looking in the `/dev` directory, there should be a device named like
`/dev/tty.usbmodemXYZ`. Where XYZ is a number, for example 101. This
is the device path that might need to be passed as `--port` when
running the client programs.

### Running a program

You can use `tkey-runapp` from
[tillitis-key1-apps](https://github.com/tillitis/tillitis-key1-apps)
to upload a program to the TKey.

```
$ tkey-runapp apps/blink/app.bin
```

This should auto-detect any attached TKeys and upload and start a very
small TKey program that blinks the LED in many different colours.

Many client programs embed the TKey program they want to use in their
own binary, auto-detect any TKeys and uploads the program
automatically. See, for instance, `tkey-ssh-agent` which embeds the
TKey program `signer`.

## Developing TKey programs

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

### Debugging TKey programs

If you're running the app on our qemu emulator we have added a debug
port on 0xfe00\_1000 (`TK1_MMIO_QEMU_DEBUG`). Anything written there
will be printed as a character by qemu on the console.

`qemu_putchar()`, `qemu_puts()`, `qemu_putinthex()`, `qemu_hexdump()`
and friends (see `apps/libcommon/lib.[ch]`) use this debug port to
print stuff.

`libcommon` is compiled with no debug output by default. Rebuild
`libcommon` without `-DNODEBUG` to get the debug output.

The emulator can output some memory access (and other) logs. You can
add `-d guest_errors` to the qemu command line to make QEMU send these
to stderr.

You can also use the qemu monitor for debugging, e.g. `info
registers`, or run qemu with `-d in_asm` or `-d trace:riscv_trap` for
tracing.

TODO Give examples on how to use gdb with qemu.

## Developing client programs

TODO write about the Go modules

- [Go doc for github.com/tillitis/tillitis-key1-apps/tk1](https://pkg.go.dev/github.com/tillitis/tillitis-key1-apps/tk1)
- [Go doc for
github.com/tillitis/tillitis-key1-apps/tk1sign](https://pkg.go.dev/github.com/tillitis/tillitis-key1-apps/tk1sign)
