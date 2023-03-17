---
title: Tools & libraries
weight: 2
---

# Tools & Libraries

## Toolchain

To build a TKey program or client program, you need at least the 
following packages installed:
- `clang`
- `llvm`
- `lld`
- `golang`

Version 15 or later of LLVM/Clang is required (with riscv32
support and the Zmmul extension, `-march=rv32iczmmul`). Packages on
Ubuntu 22.10 (Kinetic) are known to work.

On Ubuntu, you can install the required packages with the following command:

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

## Toolchain Container

As a convenience, Tillitis provides a Docker/Podman image which has all these
packages already installed. You can use the following command to fetch the image:

```
$ podman pull ghcr.io/tillitis/tkey-builder:latest
```

TODO Add example of running.

## QEMU

Tillitis provides a TKey emulator based on QEMU. 

Go to the `tk1` branch at [qemu](https://github.com/tillitis/qemu)
to fetch the emulator and then build it, or execute the 
following commands:

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

Then execute the following commands to run the emulator and 
pass the built firmware to "-bios":

```
$ /path/to/qemu/build/qemu-system-riscv32 -nographic -M tk1,fifo=chrid -bios firmware.elf \
  -chardev pty,id=chrid
```

VB: When and how is QEMU telling this? With the command above?

QEMU tells you what serial port it is using, for instance `/dev/pts/1`.
This is what you need to set as `--port` when running the client
programs.

## Software Development Kit, or, Building our TKey Programs

VB: I think the title is unclear.

There is not yet any stand-alone Software Development Kit (SDK). Instead, Tillitis
provides SDK examples in the [tillitis-key1-apps](https://github.com/tillitis/tillitis-key1-apps)
GitHub repository.

Execute the following command to clone our TKey program repository:

```
$ git clone https://github.com/tillitis/tillitis-key1-apps.git
$ cd tillitis-key1-apps
```

VB: I think "everything" should be clarified, at least by saying something like 
"all of the packages above" instead. 

Execute the following command to build everything:

```
$ make
```

If your available `objcopy` is anything other than the default
`llvm-objcopy`, then define `OBJCOPY` to whatever they're called on
your system.

TKey programs can run both on TKey and in the QEMU emulator. 
In both cases, the client program (for example `tkey-ssh-agent`)
talks to the TKey program over a serial port, virtual or real.
There is a separate section below that explains how to run
programs in QEMU.

## Running TKey Programs

To run TKey, insert it in a USB port on a computer. If the TKey status indicator LED 
is white, then TKey has been programmed with the standard FPGA bitstream (including the firmware). 
If the status indicator LED is not white, it is unprovisioned. For instructions on how to do the
initial programming of a TKey, see 
[quickstart.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/quickstart.md)
(in the tillitis-key1 repository).

The examples below refer to files in the
[tillitis-key1-apps repository](https://github.com/tillitis/tillitis-key1-apps).

### Users on Linux

Running `lsusb` should list the USB stick as `1207:8887 Tillitis
MTA1-USB-V1`. On Linux, the TKey's serial port device path is
typically `/dev/ttyACM0` (but it may end with another digit, if you
have other devices plugged in already). The client programs try to
auto-detect TKeys, but if more than one TKey is found you need to choose
one using the `--port` flag.

Your current Linux user must have read and write access 
to the serial port. One way the access is by installing the provided
`system/60-tkey.rules` in `/etc/udev/rules.d/` and running `udevadm
control --reload`. When a TKey is plugged in its device path (like
`/dev/ttyACM0`), it should be accessible by anyone logged in on the
console (see `loginctl`). 
Another way to get the access is by becoming a member of the 
`dialout` group that owns the serial port. 
On Ubuntu that group is `dialout`, and you can do it like this:

```
$ id -un
exampleuser
$ ls -l /dev/ttyACM0
crw-rw---- 1 root dialout 166, 0 Sep 16 08:20 /dev/ttyACM0
$ sudo usermod -a -G dialout exampleuser
```

For the change to take effect, you need to log out from your
system and then log back in again or run the command `newgrp dialout`
in the terminal that you are working in.

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

VB: Why "might" above?

### Running a TKey Program

You can use `tkey-runapp` from
[tillitis-key1-apps](https://github.com/tillitis/tillitis-key1-apps)
to upload a TKey program to the TKey.

```
$ tkey-runapp apps/blink/app.bin
```

This should auto-detect any attached TKeys, upload, and start a tiny
TKey program that blinks the LED in many different colors.

Many client programs embed the TKey program they want to use in their
own binary, auto-detect any TKeys, and uploads the TKey program
automatically to the TKey. See, for instance, `tkey-ssh-agent` which embeds the
TKey program `signer`.

## Developing TKey Programs

TKey programs and libraries are kept under the `apps` directory. A C
runtime is provided as `apps/libcrt0/libcrt0.a` which you can link
your C programs with.

### Memory

RAM starts at 0x4000\_0000 and ends at 0x4002\_0000. The TKey program is
loaded by firmware at 0x4000\_0000. The assembler program in
`apps/libcrt0/crt0.S` sets up a 28 kB stack at the top of the RAM.

VB: "Yourself" - is it the program you mean?

There are no heap allocation functions, no `malloc()` and friends. You
can access memory directly yourself. Tillitis provides `APP_ADDR` and
`APP_SIZE` so the loaded TKey program knows its size and where it is loaded.

Special memory areas for memory mapped hardware functions are
available at base 0xc000\_0000 and an offset. See [the memory
map](../memory/). and the include file `tk1_mem.h`.

### Debugging TKey Programs

If you run a TKey program in our QEMU emulator, there is a debug
port on 0xfe00\_1000 (`TK1_MMIO_QEMU_DEBUG`). Anything written there
is printed as a character by QEMU on the console.

`qemu_putchar()`, `qemu_puts()`, `qemu_putinthex()`, `qemu_hexdump()`
and friends (see `apps/libcommon/lib.[ch]`) use this debug port to
print an output.

`libcommon` is compiled with no debug output by default. Rebuild
`libcommon` without `-DNODEBUG` to get the debug output.

The emulator can output some memory access (and other) logs. You can
add `-d guest_errors` to the qemu command line to make QEMU send these
to `stderr`.

You can also use the QEMU monitor for debugging, for example, `info
registers`, or run QEMU with `-d in_asm` or `-d trace:riscv_trap` for
tracing.

TODO Give examples on how to use gdb with QEMU.

## Developing Client Programs

TODO write about the Go modules

- [Go doc for github.com/tillitis/tillitis-key1-apps/tk1](https://pkg.go.dev/github.com/tillitis/tillitis-key1-apps/tk1)
- [Go doc for
github.com/tillitis/tillitis-key1-apps/tk1sign](https://pkg.go.dev/github.com/tillitis/tillitis-key1-apps/tk1sign)

