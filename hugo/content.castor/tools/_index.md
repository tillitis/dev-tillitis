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

To create applications you need at least `make`, `clang`, `llvm`,
`lld`, and `go` or `golang` packages installed. Version 15 or later of
LLVM/Clang is required (with riscv32 support and the Zmmul extension,
`-march=rv32iczmmul`).


### Linux

Many Linux distributions have the above as packages. Ubuntu 23.04 is
known to work.

### macOS

First you need the Xcode Command Line Tools installed.

```
xcode-select --install
```

This will give you `make` and other useful tools for development. Even
if macOS provides `llvm` it does not seem to support our target,
`riscv32-unknown-none-elf`, so we recommend to also installing `llvm`,
among other packages, with [Homebrew](https://brew.sh/):

```
brew install llvm go
```

One caveat for llvm is that it is "keg-only", which means it was not
symlinked into `/opt/homebrew`, and we then need to do it ourselves.

The easiest way is to add

```
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
```

to your your `.zshrc` or equivalent. The key is that we add `llvm`
from brew in `PATH` before `llvm` provided by macOS. Just remember
that if you use `llvm` provided by macOS for other projetcs this can
create issues. Another way would be to explicitly specify which to use
in the makefiles.

### Windows

The easiest way to install the required packages is through the
package manager [Chocolatey](https://community.chocolatey.org/). After
installing Chocolatey, run Powershell (version 3 or higher) as an
administrator, and install the necessary packages using the following
command:

```
choco install make llvm clang go
```

## Toolchain container `tkey-builder`

We provide a container image which has all the above packages and
tools already installed for use with Podman or Docker.

You should probably always use the image with the tag "latest", but if
you're trying to do a reproducible build you should of course use the
tag possibly mentioned in the release.

{{< tabs >}}
{{% tab "Linux" %}}

This assumes a working rootless Podman. On Ubuntu 22.10, running

```
sudo apt install podman rootlesskit slirp4netns
```

should be enough to get you a working Podman setup.

{{% /tab %}}
{{% tab "macOS" %}}
Podman for macOS is distributed using brew.

```
brew install podman
```

Next, create and start your first Podman machine:

```
podman machine init
podman machine start
```

You can then verify the installation information using:

```
podman info
```

It is also possible to use binaries or a pkginstaller on Podman's
[Github release page](https://github.com/containers/podman/releases).

{{% /tab %}}
{{% tab "Windows" %}}
To install on Windows is a bit more complicate, follow this link for
comprehensive instructions:

<https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md>

{{% /tab %}}
{{< /tabs >}}

You can use the following command to fetch the image:

```
podman pull ghcr.io/tillitis/tkey-builder
```

**Note well**: This image is really large (~ 2 GiB) because it also
contains all the tools necessary to build the FPGA bitstream and the
firmware.

## Device libraries

Libraries for development of TKey device apps are available in:

<https://github.com/tillitis/tkey-libs>

Pre-compiled versions are available under:

<https://github.com/tillitis/tkey-libs/releases>

Unpack the tar file somewhere and point clang to where they are,
typically with `-L tkey-libs` and `-I tkey-libs/include.`

In many device app projects it will be sufficient to set `LIBDIR`:

```
make LIBDIR=/path/to/tkey-libs
```

Note that your `lld` might complain if it's not the same version that
was used to produce the libraries. You might want to build the
libraries yourself if that happens, or use the tkey-builder container
image.

To build tkey-libs, typically you just:

```
git clone https://github.com/tillitis/tkey-libs.git
cd tkey-libs
make
```

or

```
make podman
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
  ed25519 signing tool. [Go
  doc](https://pkg.go.dev/github.com/tillitis/tkeysign).

## Building applications

### Building with host tools

Most of the apps listed under [projects](/projects/) comes with a
Makefile and can be built with:

```
make
```

If they have complex dependencies they might come with a `build.sh`
script to clone and build the dependencies first.

If tkey-libs is cloned and built somewhere other than in the default
directory called tkey-libs, next to the app directory that needs it,
you need to specify the path to it, as follows:

```
make LIBDIR=../tkey-libs-main
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
make podman
```

Or use podman directly if you haven't got `make` installed, typically
specifying where your `tkey-libs` are:

```
podman run --rm --mount type=bind,source=.,target=/src --mount type=bind,source=../tkey-libs,target=/tkey-libs -w /src -it ghcr.io/tillitis/tkey-builder make -j
```

## QEMU Emulator

Tillitis provides a TKey emulator based on QEMU.

### Running with OCI image

The easiest way to run the TKey emulator is to use our OCI image (~120
MiB). It currently only works on a Linux system (specifically, it does
not work when containers are run in Podman's virtual machine, which is
required on macOS and Windows). So for non-Linux users, see [Building
QEMU](/tools/#building-qemu).

```
ghcr.io/tillitis/tkey-qemu-tk1-23.03.1:latest
```

The OCI image also contains the TKey firmware and starts QEMU with it.

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

TODO: Mention the scripts to use to handle the USB Mode Protocol and
attach a soft HID device.

### Running manually

If you want to run QEMU without using the OCI image you have to first
build our QEMU fork (see below).

Build the TKey firmware:

```
git clone https://github.com/tillitis/tillitis-key1
cd tillitis-key1/hw/application_fpga
make qemu_firmware.elf
```

Generate a flash image file `flash.bin` using the
`application_fpga/tools/tkeyimage` and copy it to where you are going
to run QEMU.

Run QEMU using the `qemu_firmware.elf` file you built and the
`flash.bin` file you generated:

```
$ /path/to/qemu/build/qemu-system-riscv32  -nographic -M
tk1-castor,fifo=chrid -bios qemu_firmware.elf -chardev pty,id=chrid -s
-d guest_errors -drive file=flash.bin,if=mtd,format=raw,index=0
```

If you're trying to emulate the earlier releases of the TKey, use `-M
tk1` and drop the `-drive` flag.

TODO: Move the above to Bellatrix part?

QEMU tells you which serial port it's using, for instance
`/dev/pts/1`. This is what you need to set with `--port` when running
a client application.

### QEMU on macOS

Note that on macOS you need to add `--speed 9600` on the client apps
when you use the QEMU pty.

### Building QEMU

To build QEMU, fetch and build the `tk1` branch in our [qemu
repository](https://github.com/tillitis/qemu):

```
git clone -b tk1 https://github.com/tillitis/qemu
mkdir qemu/build
cd qemu/build
../configure --target-list=riscv32-softmmu --disable-werror --without-default-features
make -j $(nproc) qemu-system-riscv32
```

(Built with warnings-as-errors disabled, see [this
issue](https://github.com/tillitis/qemu/issues/3).)
