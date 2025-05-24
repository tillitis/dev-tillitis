---
title: Program SPI flash
weight: 3
---

# Program the SPI flash of the TKey

Programming the SPI flash of the TKey makes it possible to develop and
test different FPGA configurations and firmwares while
reprogramming multiple times.

{{< hint warning >}}
The entire content of the SPI flash can be read by anyone who has
access to the TKey. If you want a secure, locked-down version of TKey,
you should instead program the [NVCM](unlocked/nvcm).
{{< /hint >}}

Programming the SPI flash is done in two steps:
1. Download tools.
2. Program the SPI flash.

{{< tabs "flash SPI" >}}
{{% tab "Linux" %}}

The SPI flash can be programmed either in the container or natively.
If you are using our container you can go directly to step 2.

## 1. Download tools

When programming the SPI flash we are using `iceprog` from our fork of
the Yosys icestorm project:

https://github.com/tillitis/icestorm/

If you want to use `iceprog` natively you will need to compile only
`icestorm/iceprog` in the "interfaces" branch with:

```
make PROGRAM_PREFIX=tillitis-
```

By default it installs with `make install` in `/usr/local/bin`.

Our OCI image [tkey-builder](https://ghcr.io/tillitis/tkey-builder)
includes our version of `iceprog` for use with Podman.

## 2. Program SPI flash

First ensure that the TKey is inserted into the TKey Programmer and
that the programmer is inserted into your computer.

Note for this to work, you need permission to speak to the raw USB
device of the TKey Programmer. See [Linux device
permissions](tp1/#linux-permissions).

### Podman

If you're on an SELinux system you might need to run this first to be
able to access USB devices from the container:

```
setsebool container_use_devices=true
```

The simplest way is to use the make target

```
make -C contrib flash
```

which will start the container and program the SPI flash directly. If
this succeeds you are done and can skip the remaining steps.
Otherwise start the container using:

```
podman run --rm --device /dev/bus/usb/$(lsusb | grep -m 1 1209:8886 | awk '{ printf "%s/%s", $2, substr($4,1,3) }') -v .:/build:Z -w /build -it ghcr.io/tillitis/tkey-builder /usr/bin/bash
```

### Programming

In a native shell or your container shell:

```
tillitis-iceprog hw/application_fpga/application_fpga.bin
```

{{< /tab >}}
{{< tab "macOS" >}}
## 1. Download tools

When programming the SPI flash we are using `iceprog` from our fork of
the Yosys icestorm project:

https://github.com/tillitis/icestorm/

If you want to use `iceprog` natively you will need to compile only
`icestorm/iceprog` in the "interfaces" branch with:

```
make PROGRAM_PREFIX=tillitis- OS=macos
```

By default it installs with `make install` in `/usr/local/bin`.

You can also download a binary release here:

https://github.com/tillitis/icestorm/releases

## 2. Program SPI flash

First ensure that the TKey is inserted into the TKey Programmer and
that the programmer is inserted into your computer.

After installing `tillitis-iceprog`, run:

```
tillitis-iceprog hw/application_fpga/application_fpga.bin
```

{{< /tab >}}
{{< tab "Windows" >}}

{{< hint info >}}
At the current state Tillitis does not provide a simple way of
programming the SPI flash from a Windows machine. We are working on a
solution.
{{< /hint >}}
When programming the SPI flash we are using `iceprog` from our fork of
the Yosys icestorm project:

https://github.com/tillitis/icestorm/

Use the "interfaces" branch.

According to Yosys it shoud be possible to compile but Tillitis has
not yet explored this solution, please see the bottom of [Yosys
download page](https://yosyshq.net/yosys/download.html) if you want to
try.

{{< /tab >}}
{{< /tabs >}}


Your Tkey is now ready to be taken out of the programmer.

{{< hint info >}}
If you plan to later lock down your Tkey by flashing the NVCM, it
is highly recommended to generate a new bitstream with a new unique
UDS and UDI.
{{< /hint >}}

Once you are done, continue to [casing](unlocked/casing) to assemble
the plastic case.
