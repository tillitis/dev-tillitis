---
title: Tools & Libraries
weight: 2
---

# Tools & Libraries

## Toolchain

To build you need at least `clang`, `llvm`, `lld`, `golang` packages
installed. Version 15 or later of LLVM/Clang is required (with riscv32
support and the Zmmul extension, `-march=rv32iczmmul`). Packages on
Ubuntu 22.10 (Kinetic) are known to work.

On Ubuntu, you can install the required packages with the following
command:

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

## Toolchain Container

As a convenience, Tillitis provides a container image which has all
these packages and tools already installed, for use with Podman or
Docker. You can use the following command to fetch the image:

```
$ podman pull ghcr.io/tillitis/tkey-builder:latest
```

To build everything in the apps repo:

```
$ git clone https://github.com/tillitis/tillitis-key1-apps
$ cd tillitis-key1-apps
$ podman run --rm --mount type=bind,source="$(pwd)",target=/src -w /src -it ghcr.io/tillitis/tkey-builder:2 make -j
```

## QEMU

Tillitis provides a TKey emulator based on QEMU.

Go to the `tk1` branch in our [qemu
repository](https://github.com/tillitis/qemu) to fetch the emulator
and then build it, or execute the following commands:

```
$ git clone -b tk1 https://github.com/tillitis/qemu
$ mkdir qemu/build
$ cd qemu/build
$ ../configure --target-list=riscv32-softmmu --disable-werror
$ make -j $(nproc)
```

(Built with warnings-as-errors disabled, see [this
issue](https://github.com/tillitis/qemu/issues/3).)

Then execute the following commands to fetch and build the firmware:

```
$ git clone https://github.com/tillitis/tillitis-key1
$ cd tillitis-key1/hw/application_fpga
$ make firmware.elf
```

Then execute the following commands to run the emulator, setting the
built firmware with the `-bios` flag:

```
$ /path/to/qemu/build/qemu-system-riscv32 -nographic -M tk1,fifo=chrid -bios firmware.elf \
  -chardev pty,id=chrid
```

In the output from QEMU it tells you which serial port it's using, for
instance `/dev/pts/1`. This is what you need to use as `--port` when
This is what you need to set with `--port` when running a client
application.

## Software Development Kit, or, Building our TKey Client and Device Apps

There is not yet any stand-alone Software Development Kit (SDK).
Instead, Tillitis provides examples in the
[tillitis-key1-apps](https://github.com/tillitis/tillitis-key1-apps)
GitHub repository.

Execute the following command to clone the repository:

```
$ git clone https://github.com/tillitis/tillitis-key1-apps.git
$ cd tillitis-key1-apps
```

Execute the following command to build all TKey client and device
applications:

```
$ make
```

If your available `objcopy` is anything other than the default
`llvm-objcopy`, then define `OBJCOPY` to whatever they're called on
your system.

TKey device applications can run both on the real hardware TKey and in
the QEMU emulator. In both cases, the client application (for example
`tkey-ssh-agent`) talks to the device app over a serial port, virtual
or real. There is a separate section below that explains how to run
device apps in QEMU.

## Running TKey apps

To run TKey, plug it into a USB port on a computer. If the TKey status
indicator LED is white, then it has been programmed with the standard
FPGA bitstream (including the firmware). If the status indicator LED
is not white, it is unprovisioned. For instructions on how to do the
initial programming of a TKey, see:
[quickstart.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/quickstart.md)
(in the tillitis-key1 repository).

The examples below refer to files in the
[tillitis-key1-apps repository](https://github.com/tillitis/tillitis-key1-apps).

### Users on Linux

Running `lsusb` should list the USB stick as `1207:8887 Tillitis
MTA1-USB-V1`. On Linux, the TKey's serial port device path is
typically `/dev/ttyACM0` (but it may end with another digit, if you
have other devices plugged in already). The client applications try to
auto-detect TKeys, but if more than one TKey is found you need to
choose one using the `--port` flag.

Your current Linux user must have read and write access to the serial
port. One way to get access is by installing the provided
`system/60-tkey.rules` in `/etc/udev/rules.d/` and running `udevadm
control --reload`. When a TKey is plugged in, its device path (like
`/dev/ttyACM0`) should be accessible by anyone logged in on the
console (see `loginctl`).

Another way to get access is by becoming a member of the group that
owns the serial port. On Ubuntu that group is `dialout`, and you can
do it like this:

```
$ id -un
exampleuser
$ ls -l /dev/ttyACM0
crw-rw---- 1 root dialout 166, 0 Sep 16 08:20 /dev/ttyACM0
$ sudo usermod -a -G dialout exampleuser
```

For the change to take effect, you need to log out from your system
and then log back in again, or run the command `newgrp dialout` in the
terminal that you are working in.

### Users on MacOS

The client apps tries to auto-detect serial ports of TKey USB sticks,
but if more than one is found you'll need to choose one using the
`--port` flag.

To find the serial ports device path manually you can do `ls -l
/dev/cu.*`. There should be a device named like `/dev/cu.usbmodemN`
(where N is a number, for example 101). This is the device path that
might need to be passed as `--port` when running the client app.

You can verify that the OS has found and enumerated the USB stick by
running:

```
ioreg -p IOUSB -w0 -l
```

There should be an entry with `"USB Vendor Name" = "Tillitis"`.

### Running a TKey Device Application

You can use `tkey-runapp` from the
[tillitis-key1-apps](https://github.com/tillitis/tillitis-key1-apps)
repository to load a device application onto the TKey.

```
$ tkey-runapp apps/blink/app.bin
```

This should auto-detect any attached TKey and load and start a very
small device application that blinks the LED in many different
colours. This should auto-detect any attached TKeys, upload, and start
a tiny device app that blinks the LED in many different colors.

Many TKey client applications embed the device app they want to use in
their own binary. They auto-detect the TKey and automatically loads
the device app onto it. See, for instance, `tkey-ssh-agent` which
embeds the device app `signer`.

## Developing TKey Device Applications

Device apps and libraries are kept under the `apps` directory. A C
runtime is provided as `apps/libcrt0/libcrt0.a` which you can link
your C programs with.

### Memory

RAM starts at 0x4000\_0000 and ends at 0x4002\_0000 (128 kiB). The
device app will be loaded by firmware at RAM start. The stack for the
app is setup to start just below the end of RAM (see
[apps/libcrt0/crt0.S](https://github.com/tillitis/tillitis-key1-apps/blob/main/apps/libcrt0/crt0.S)
in the tillitis-key1-apps repository).

There are no heap allocation functions, no `malloc()` and friends. You
can access memory directly yourself. `APP_ADDR` and `APP_SIZE` are
provided so the loaded device app knows where it's loaded and how
large it is.

Special memory areas for memory mapped hardware functions are
available at base 0xc000\_0000 and an offset. See [the memory
map](../memory/). and the include file `tk1_mem.h`.

### Debugging

If you a TKey device app in the QEMU emulator, there is a debug debug
port on 0xfe00\_1000 (`TK1_MMIO_QEMU_DEBUG`). Anything written there
is printed as a character by QEMU on the console.

`qemu_putchar()`, `qemu_puts()`, `qemu_putinthex()`, `qemu_hexdump()`
and friends (see `apps/libcommon/lib.[ch]`) use this debug port to
print out things.

`libcommon` is compiled with no debug output by default. Rebuild
`libcommon` without `-DNODEBUG` to get the debug output.

The emulator can output some memory access (and other) logs. You can
add `-d guest_errors` to the qemu command line to make QEMU send these
to stderr.

You can also use the QEMU monitor for debugging, for example, `info
registers`, or run QEMU with `-d in_asm` or `-d trace:riscv_trap` for
tracing.

TODO Give examples on how to use gdb with qemu.

## Developing TKey client applications

TODO write about the Go modules

- [Go doc for github.com/tillitis/tillitis-key1-apps/tk1](https://pkg.go.dev/github.com/tillitis/tillitis-key1-apps/tk1)
- [Go doc for
github.com/tillitis/tillitis-key1-apps/tk1sign](https://pkg.go.dev/github.com/tillitis/tillitis-key1-apps/tk1sign)
