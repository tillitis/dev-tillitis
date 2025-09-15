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

Tillitis provides a TKey emulator based on QEMU. It's still in
development for Castor but can be used with some caveats. For
instance, there is no real system call protection in the emulation
right now.

### Building QEMU

To build QEMU, fetch and build the `tk1` branch in our [qemu
repository](https://github.com/tillitis/qemu):

```
git clone -b tk1 --depth 1 https://github.com/tillitis/qemu
mkdir -p qemu/build
cd qemu/build
../configure --target-list=riscv32-softmmu --without-default-features
make -j $(nproc) qemu-system-riscv32
```

Remove `--depth 1` if you want all history.

**NOTE WELL**: 

- The tk1 branch is a development branch. There is currently no
QEMU release for Castor support.
- Currently only well supported on Linux.

### Running manually

To run QEMU you need to build the TKey firmware and the filesystem
image.

Clone the tillitis-key1 repo:

```
git clone --depth 1 https://github.com/tillitis/tillitis-key1
cd tillitis-key1/hw/application_fpga
```

(Remove `--depth 1` if you really want all history.)

Build the flash image file `flash_image.bin` that QEMU uses for storage:

```
make clean && make flash_image.bin
```

Build the TKey QEMU firmware:

```
make qemu_firmware.elf
```

Run QEMU using the `qemu_firmware.elf` and `flash_image.bin` files:

```
/path/to/qemu/build/qemu-system-riscv32 \
    -nographic \
    -M tk1-castor,fifo=chrid \
    -bios qemu_firmware.elf \
    -chardev pty,id=chrid \
    -d guest_errors \
    -drive file=flash_image.bin,if=mtd,format=raw,index=0 \
    -s
```

QEMU tells you which serial port it's using, for instance
`/dev/pts/1`.

**Please note**: Below we're using `/dev/pts/1` all the time for the
QEMU port.

Because of Castor's new USB mode protocol between the USB controller
and the FPGA you will need to use the `qemu_usb_mux.py` script below
that handles communication with `/dev/pts/1`.

There are a number of other Python scripts that can be useful,
documented below. They are available in `qemu/tools/tk1`.

### The endpoint multiplexor script

The `qemu_usb_mux.py` script creates separate PTYs for communication
with each of the CDC, FIDO, CCID, and DEBUG endpoints. When data is
sent to/received from each PTY the script handles Castor's USB mode
protocol and multiplexes/demultiplexes data.

Typical use:

1. Start QEMU in a shell. Use the `-S` flag if you want to stop the
   execution to be able able to see the char device QEMU is using.

   ```
   /path/to/qemu/build/qemu-system-riscv32 \
     -nographic \
     -M tk1-castor,fifo=chrid \
     -bios qemu_firmware.elf \
     -chardev pty,id=chrid \
     -d guest_errors \
     -drive file=flash_image.bin,if=mtd,format=raw,index=0 \
     -s \
     -S
   char device redirected to /dev/pts/1 (label chrid)
   HTIF not enabled (no debug output from firmware)
   QEMU X.Y.Z monitor - type 'help' for more information
   (qemu)
   ```

   Note the serial port used, for instance `/dev/pts/1`.

2. Start `qemu_usb_mux.py` in another shell with QEMU's char device as
   argument:

    ```
    python qemu_usb_mux.py /dev/pts/1
    CDC PTY created at: /dev/pts/2
    FIDO PTY created at: /dev/pts/3
    CCID PTY created at: /dev/pts/4
    DEBUG PTY created at: /dev/pts/5
    ```

3. Start the QEMU execution if it was stopped.

4. Load app into QEMU over the CDC PTY:

    ```
    tkey-runapp --port /dev/pts/2 app.bin
    ```

### Emulating a FIDO token

The `fido2_token_emulator.py` script can be used with the Linux UHID
framework (a user-space I/O driver support for HID subsystem) to
create a hidraw device that emulates a FIDO2 token. The FIDO2 data
sent to the hidraw device will be output on `/dev/uhid` and is then
sent in to the FIDO PTY created above. Access to `/dev/uhid` is
restricted to root so use sudo when starting `fido2_token_emulator.py`
or change the mode of the `/dev/uhid` file before running.

Typical use:

1. Start QEMU as shown above.

2. Start `qemu_usb_mux.py` as shown above.

3. Start the QEMU execution if it was stopped.

4. Load FIDO2 app for debugging to QEMU to the CDC PTY:

    ```
    tkey-runapp --port /dev/pts/2 app.bin
    ```

5. Start `fido2_token_emulator.py` with the FIDO PTY:

   ```
   sudo python fido2_token_emulator.py /dev/pts/3
   ```

   If you're uncomfortable running the script as root you can do:

   ```
   sudo chgrp dialout /dev/uhid
   sudo chmod 660 /dev/uhid
   python fido2_token_emulator.py /dev/pts/3
   ```

   Replace "dialout" to the group you're in and want to allow to
   create HID devices.

6. Check the created FIDO2 device with:

    ```
    fido2-token -L
    ```

7. Try communicating with the FIDO2 device:

    ```
    fido2-token -I /dev/hidrawX
    ```

### Using the loopback device app

We have developed a small loopback device app. It is used when you
want to loop the I/O through a real TKey but you're actually running
your own device app somewhere else, like under emulation in QEMU. The
use case is for example to test how Windows would interact with a
physical TKey that present itself as a CDC device, FIDO token or CCID
device.

When running the loopback device app, all data coming in on the USB
endpoints CDC, FIDO, or CCID on the physical TKey will be sent out on
the DEBUG endpoint back to the client. If you want to, you can listen
on the DEBUG endpoint, forward the data over a network to a QEMU on
another computer.

First build the `loopbackapp.bin` device app:

```
git clone --depth 1 https://github.com/tillitis/tillitis-key1
cd tillitis-key1/hw/application_fpga/apps
make
```

Helper scripts:

- `tkey_to_udp_linux.py`: Sends data from the DEBUG endpoint over the
  network. For Linux hosts.

- `tkey_to_udp_win_mac.py`: Sends data from the DEBUG endpoint over the
  network. For Windows and macOS.

- `hid.py`: USB HID helper script.

#### Example 1: Debugging FIDO2 app locally on Linux

Typical use:

1. Start QEMU as shown above.

2. Start `qemu_usb_mux.py` as shown above.

3. Start the QEMU execution if it was stopped.

4. Load FIDO2 app for debugging to QEMU with the CDC PTY (note what
   `qemu_usb_mux.py` says is the CDC PTY):

    ```
    tkey-runapp --port /dev/pts/2 app.bin
    ```

5. Shutdown the `qemu_usb_mux.py` script.

6. Start the UDP listening server to forward data to QEMU. Localhost
   is used as an example here but it can be the real network and send
   to another host. Use the QEMU PTY from when you started QEMU, in
   this example `/dev/pts/1`:

    ```
    python udp_to_qemu_linux.py \
        --pty /dev/pts/1 \
        --listen-ip 127.0.0.1 \
        --listen-port 5678 \
        --verbose
    ```

7. Insert the TKey in the physical computer and load
   `loopbackapp.bin`. Make sure you have added the Linux udev rules
   for TKey from [Linux Users](/devapp/#linux-users).

    ```
    tkey-runapp loopbackapp.bin
    ```

8. Start TKey listener to forward data from computer with TKey to
   server (dest-ip, dest-port) with QEMU. Make sure you have installed
   the `hidapi` library from https://github.com/libusb/hidapi For
   Ubuntu: `sudo apt-get install libhidapi-dev`.

   ```
   python tkey_to_udp_linux.py \
       --dest-ip 127.0.0.1 \
       --dest-port 5678 \
       --listen-ip 127.0.0.1 \
       --listen-port 5679 \
       --verbose
   ```

9. Check FIDO2 device:

    ```
    fido2-token -L
    ```

10. Try communicating with the FIDO2 device and see the data
    forwarded to QEMU.

    ```
    fido2-token -I /dev/hidrawX
    ```

#### Example 2: Debugging FIDO2 using Windows on VirtualBox

In this scenario we're running Windows in VirtualBox on a Linux host
with QEMU emulating the TKey on Linux computer, not necessarily the
same Linux host as is running the VirtualBox.

Typical use:

1. Start QEMU on Linux computer as shown above.

2. Start `qemu_usb_mux.py` as shown above.

3. Start the QEMU execution if it was stopped.

4. Load FIDO2 app for debugging to QEMU using the CDC PTY:

    ```
    tkey-runapp --port /dev/pts/2 app.bin
    ```

5. Shutdown `qemu_usb_mux.py` script.

6. Start a UDP listening server on the Linux host computer to forward
   data to QEMU. In this example the Linux host IP address is
   192.168.100.10. Since we are going to do port forwarding to
   Windows, the IP address needs to be routable and can't be
   localhost.

    ```
    python udp_to_qemu_linux.py \
        --pty /dev/pts/1 \
        --listen-ip 192.168.100.10 \
        --listen-port 5678 \
        --verbose
   ```

7. Start a VirtualBox with Windows.

   Add a port forward to the VirtualBox Windows instance:

   - Right click on Windows instance -> Settings -> Network
   - Enable network adapter
   - Set "Attached to: NAT"
   - Go "Advanced" -> Port Forwarding
   - Add new rule
   - Select UDP as protocol
   - Set Host Port to 5679
   - Set Guest Port to 5679

8. On Windows, install the following.

   - Python3 from https://www.python.org/downloads/windows/

     Make sure to select to install "py launcher" and "Add Python to
     environment variables".

   - hidapi version from https://github.com/libusb/hidapi/releases/

     Copy hidapi.dll and hidapi.lib to C:\Windows\System32\

   - libfido2 (fido2-token) from

     https://developers.yubico.com/libfido2/Releases/

9. Get the main IP address of your Windows computer, for example: 10.0.2.15

10. Connect the TKey to the VirtualBox environment in the USB
    settings, make sure Windows says:

    "Device is ready, Tillitis TKEY-USB-V2 is set up and ready to go"

11. On Windows, start the `tkey_to_udp_win_mac.py` script, using a
    PowerShell with `Admin` privliges.

     ```
     python tkey_to_udp_win_mac.py \
         --dest-ip 192.168.100.10 \
         --dest-port 5678 \
         --listen-ip 10.0.2.15 \
         --listen-port 5679 \
         --verbose
     ```

12. Using a PowerShell with `Admin` privliges, check that the TKey's
    FIDO interface is visible with fido2-token.exe from the libfido2
    library.

     ```
     .\fido2-token.exe -L
     \\?\hid#vid_1209&pid_8885&mi_02#7&180c60dc&0&0000#{4d1e55b2-f16f-11cf-88cb-001111000030}: vendor=0x1209, product=0x8885 (Tillitis FIDO)
      ```

13. Try communicating with the FIDO2 device and see the data forwarded to
    QEMU.

     ```
     .\fido2-token.exe -I "\\?\hid#vid_1209&pid_8885&mi_02#7&180c60dc&0&0000#{4d1e55b2-f16f-11cf-88cb-001111000030}
     ```
