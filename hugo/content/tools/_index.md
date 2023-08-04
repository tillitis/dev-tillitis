---
title: Tools & libraries
weight: 2
---

# Tools & libraries

## Introduction

To build applications, you can either use our OCI images or use native
tools on your dev box.

If you want your device applications not to change, which as you know
also means changing the application's CDI as explained in [the
introduction](intro/), it might be better to use the OCI images. At
the very least you want to be sure that the versions of the compiler
and other tools you use stay the same. Perhaps pin those packages if
you don't want to use containers?

## Host toolchain

To create applications you need at least `clang`, `llvm`, `lld`,
`golang` packages installed. Version 15 or later of LLVM/Clang is
required (with riscv32 support and the Zmmul extension,
`-march=rv32iczmmul`). Packages on Ubuntu 22.10 (Kinetic) are known to
work.

On Ubuntu, you can install the required packages with the following
command:

```
$ sudo apt install build-essential clang lld llvm bison flex libreadline-dev \
                 gawk tcl-dev libffi-dev git mercurial graphviz \
                 xdot pkg-config python3 libftdi-dev \
                 python3-dev libeigen3-dev \
                 libboost-dev libboost-filesystem-dev \
                 libboost-thread-dev libboost-program-options-dev \
                 libboost-iostreams-dev cmake libusb-1.0-0-dev \
                 ninja-build libglib2.0-dev libpixman-1-dev \
                 golang clang-format
```

On Windows, the easiest way to install the required packages is
through the package manager
[Chocolatey](https://community.chocolatey.org/). After installing
Chocolatey, run Powershell (version 3 or higher) as an administrator,
and install the necessary packages using the following command:
```
$ choco install make llvm clang go
```

## Toolchain container `tkey-builder`

We provide a container image which has all the above packages and
tools already installed for use with Podman or Docker.

This assumes a working rootless Podman. On Ubuntu 22.10, running

```
$ sudo apt install podman rootlesskit slirp4netns
```

should be enough to get you a working Podman setup.

You can use the following command to fetch the image:

```
$ podman pull ghcr.io/tillitis/tkey-builder:2
```

**Note well**: This image is really large (~ 2 GiB) because it also
contains all the tools necessary to build the FPGA bitstream and the
firmware.

## Device libraries

Libraries for development of TKey device apps are available in:

https://github.com/tillitis/tkey-libs

Build the tkey-libs first, typically just:

```
$ git clone https://github.com/tillitis/tkey-libs.git
$ cd tkey-libs
$ make
```

or

```
$ make podman
```

if you have Podman installed.

## Client libraries

We provide some Go packages to help in developing client applications.
What we call "client" is the computer or mobile device you insert your
TKey into.

- [github.com/tillitis/tkeyclient](https://github.com/tillitis/tkeyclient):
  Contains functions to connect to, load and start a device
  application on the TKey. [Go
  doc](https://pkg.go.dev/github.com/tillitis/tkeyclient).
- [github.com/tillitis/tkeyutil](https://github.com/tillitis/tkeyutil):
  Utility functions input the USS or send notifications. [Go
  doc](https://pkg.go.dev/github.com/tillitis/tkeyutil).
- [github.com/tillitis/tkeysign](https://github.com/tillitis/tkeysign):
  Contains functions to communicate with the `signer` device app, an
  ed25519 signing oracle. [Go
  doc](https://pkg.go.dev/github.com/tillitis/tkeysign).

## Building applications

### Building with host tools

Most of the apps listed under [projects](/projects/) comes with a
Makefile and can be built with:

```
$ make
```

If they have complex dependencies they might come with a `build.sh`
script to clone and build the dependencies first.

If tkey-libs is cloned and built somewhere other than in the default
directory called tkey-libs, next to the app directory that needs it,
you need to specify the path to it, as follows:

```
$ make LIBDIR=../tkey-libs-main
```

If the `objcopy` binary on your system is anything other than the
default `llvm-objcopy`, define `OBJCOPY` to whatever it is called on
your system.

TKey device applications can run both on the real hardware TKey and in
the QEMU emulator. In both cases, the client application (for example
`tkey-ssh-agent`) talks to the device app over a serial port.  There
is a separate section below that explains how to run it in QEMU.

### Building with `tkey-builder`

Most of the [projects](/projects/) come with a `podman` target:

```
$ make podman
```

Or use podman directly if you haven't got `make` installed, typically
specifying where your `tkey-libs` are:

```
$ podman run --rm --mount type=bind,source=.,target=/src --mount type=bind,source=../tkey-libs,target=/tkey-libs -w /src -it ghcr.io/tillitis/tkey-builder:1 make -j
```
## QEMU Emulator

Tillitis provides a TKey emulator based on QEMU.

The easiest way to run the TKey emulator is to use our OCI image (~120
MiB). It currently only works on a Linux system (specifically, it does
not work when containers are run in Podman's virtual machine, which is
required on macOS and Windows). So for non-Linux users, see [Building
QEMU](/tools/#building-qemu).

```
ghcr.io/tillitis/tkey-qemu-tk1-23.03.1:latest
```

We provide a script `run-tkey-qemu` that runs this image and binds the
serial port to a pty called `tkey-qemu-pty` in the current directory.

You can find `run-tkey-qemu` in the
[tkey-devtools](https://github.com/tillitis/tkey-devtools) repo. It
assumes a working rootless Podman setup and `socat` installed. On
Ubuntu 22.10, running `apt install podman rootlesskit slirp4netns
socat` should be enough. Then you can just run the script like:

```
./run-tkey-qemu
```

This will let you run client apps with `--port ./tkey-qemu-pty` and it
will find the running emulator.

### QEMU on macOS

Note that on macOS you need to add `--speed 9600` on the client apps
when you use the QEMU pty.

### Building QEMU

To build QEMU, fetch and build the `tk1` branch in our [qemu
repository](https://github.com/tillitis/qemu):

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
