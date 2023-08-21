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
{{< tab "Linux" >}}

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

### Podman

Make sure you have the `tkey-builder` container running as outlined in
[Start the container](unlocked/build/#start-container).

### Programming

In a native shell or your container shell:

```
cd hw/application_fpga
tillitis-iceprog application_fpga.bin
```

Note that your own user need permission to speak to the raw USB device
of the TKey Programmer. See [Linux device
permissions](tp1/#linux-permissions).

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
cd hw/application_fpga
tillitis-iceprog application_fpga.bin
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
