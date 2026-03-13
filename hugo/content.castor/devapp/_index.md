---
title: Developing
weight: 3
---

# Developing

## Device Applications

Consider this C program:

```c
#include <stdint.h>
#include <tkey/led.h>

#define SLEEPTIME 100000

void sleep(uint32_t n)
{
	for (volatile int i = 0; i < n; i++);
}

int main(void)
{
	for (;;) {
		led_set(LED_RED);
		sleep(SLEEPTIME);
		led_set(LED_GREEN);
		sleep(SLEEPTIME);
		led_set(LED_BLUE);
		sleep(SLEEPTIME);
	}
}
```

To get this to work you will need our header files and to link with
`libcrt0` C runtime, otherwise your program won't even reach `main()`,
and `libcommon` to get the `led_set()` function.

Header files and libraries are available in our repository
[tkey-libs](https://github.com/tillitis/tkey-libs) as mentioned in
[Tools & libraries](/tools/#device-libraries).

We also provide a linker script there in `apps.lds` which shows the
linker the memory layout.

Minimal compilation of the above program would look something like
this if `tkey-libs` is in the directory next to this one:

```
clang -g -target riscv32-unknown-none-elf -march=rv32iczmmul -mabi=ilp32 \
  -mcmodel=medany -static -std=gnu99 -O2 -ffast-math -fno-common \
  -fno-builtin-printf -fno-builtin-putchar -nostdlib -mno-relax -flto \
  -Wall -Werror=implicit-function-declaration \
  -I ../tkey-libs/include \
  -I ../tkey-libs -c -o rgb.o rgb.c

clang -g -target riscv32-unknown-none-elf -march=rv32iczmmul -mabi=ilp32 \
  -mcmodel=medany -static -ffast-math -fno-common -nostdlib \
  -T ../tkey-libs/app.lds \
  -L ../tkey-libs -lcrt0 -lcommon \
  -I ../tkey-libs -o rgb.elf rgb.o

llvm-objcopy --input-target=elf32-littleriscv --output-target=binary rgb.elf rgb.bin
```

Now you have `rgb.bin` which you can load into a TKey with
`tkey-runapp` (see [Running TKey apps](/devapp/#running-tkey-apps) below).

To make development easier a sample Makefile is provided in
[tkey-libs/example-app](https://github.com/tillitis/tkey-libs/tree/main/example-app).

### Memory

RAM starts at 0x4000\_0000 and ends at 0x4000\_ffff (128 kiB). The
device app will be loaded by firmware at the start of RAM. The stack
for the app is set up by the C runtime `libcrt0` to start just below
the end of RAM and grows downwards. A larger app comes at the tradeoff
of having a smaller stack.

There are no heap allocation functions, no `malloc()` and friends. You
can access memory directly yourself. `APP_ADDR` and `APP_SIZE` are
provided so the loaded device app knows where it's loaded and how
large it is.

Special memory areas for memory mapped hardware functions are
available at base 0xc000\_0000 and an offset. Please try to use the
helper functions in [tkey-libs](https://github.com/tillitis/tkey-libs)
before accessing hardware function addresses directly.

See [the memory map](/memory/) for an overview, the [the hardware
description](/hw/) for how to use, and the header file `tk1_mem.h` in
tkey-libs.

## Running TKey apps

To run the TKey, plug it into a USB port on a computer. If the TKey
status indicator LED turns first white, then blue, it has been
programmed with the standard FPGA bitstream (including the firmware),
and then immediately starts the pre-loaded app.

If the status indicator LED is not white it might be an unprovisioned
TKey Unlocked. For instructions on how to do the initial programming
of an unprovisioned TKey, see: [TKey Unlocked](/unlocked).

### Linux Users

Running `lsusb` should list the USB stick as `1209:8885 Generic
Tillitis TKEY-USB-V2`. On Linux, the TKey's serial port device path is
typically `/dev/ttyACM0` (but it may end with another digit, if you
have other devices plugged in already). The client applications try to
auto-detect TKeys, but if more than one TKey is found you need to
choose one using the `--port` flag.

You must have read and write access to the serial port. One way to get
access as your ordinary user is by installing a udev rule like this:

```
# Mark Tillitis TKey as a security token. /usr/lib/udev/rules.d/70-uaccess.rules
# will add TAG "uaccess", which will result in file ACLs so that local user
# (see loginctl) can read/write to the serial port in /dev.
ATTRS{idVendor}=="1209", ATTRS{idProduct}=="8885", ENV{ID_SECURITY_TOKEN}="1"
```

Put this in `/etc/udev/rules.d/60-tkey.rules` and run `udevadm control
--reload`. When a TKey is plugged in, its device path (like
`/dev/ttyACM0`) should now be accessible by anyone logged in on the
console (see `loginctl`).

Another way to get access is by becoming a member of the group that
owns serial ports, on some systems with default udev rules for USB CDC
ACM devices that come and go. On Ubuntu that group is `dialout`, and
you can do it like this:

```
$ id -un
exampleuser
$ ls -l /dev/ttyACM0
crw-rw---- 1 root dialout 166, 0 Sep 16 08:20 /dev/ttyACM0
$ sudo usermod -a -G dialout exampleuser
```

For the change to take effect, you need to either log out and login
again, or run the command `newgrp dialout` in the terminal that you
are working in.

### macOS Users

The client apps tries to auto-detect the serial port of the TKey.
If more than one serial port is found, use the `--port` flag to
select the appropriate one.

To find the serial ports device path manually you can do `ls -l
/dev/cu.*`. There should be a device named `/dev/cu.usbmodemN`
(where N is a number, for example 101). This is the device path that
might need to be passed as `--port` when running the client app.

You can verify that the OS has found and enumerated the TKey by
running:

```
ioreg -p IOUSB -w0 -l
```

There should be an entry with `"USB Vendor Name" = "Tillitis"`.

### Windows Users

Client apps will automatically detect the TKey serial port, unless the
`--port` flag is used.

If required, however, finding the TKey serial port can be done using
Powershell, with the command:

```
Get-PnpDevice -PresentOnly | Where-Object { $_.InstanceId -match '^USB' }
```

The TKey will be listed under the class: 'Ports', with the name 'USB
Serial Device'.

Alternatively, one can also find the device in Device Manager, under
'Ports (COM & LPT)'. This will also show the status of the TKey.

### Running a TKey Device Application

Most client applications embed the device app in their own binary to
be able to automatically load the device app. This is the recommended
approach for a smooth expereince when using the TKey. See, for
instance, `tkey-ssh-agent` which embeds the device app `signer`.

For development or simply loading a device app onto a TKey the
`tkey-runapp` from the
[tkey-devtools](https://github.com/tillitis/tkey-devtools) repository
can be used.

```
tkey-runapp rgb.bin
```

This should auto-detect any attached TKeys, upload, and start a tiny
device app that blinks the LED as described under [Device
Applications](/devapp/#device-applications).

### Debugging

If you run a TKey device app in the [QEMU emulator](/tools/#qemu-emulator)
there is a debug port on 0xfe00\_1000 (`TK1_MMIO_QEMU_DEBUG`).
Anything written there is printed as a character by QEMU on stdout.

`debug_putchar()`, `debug_puts()`, `debug_putinthex()`,
`debug_hexdump()` and friends (see `include/tkey/debug.h` in
`tkey-libs`) use this debug port to print out things.

If you want to use this you should define `QEMU_DEBUG` when including
`tkey/debug.h`. If you don't define `QEMU_DEBUG` the functions are
void macros so you can still use them but they won't generate any
code. Typically you would have something like this among your `CFLAGS`
in your Makefile:

```
CFLAGS = ... \
#   -D QEMU_DEBUG
```

and uncomment it and recompile when you want to run an app with debug
output using the QEMU debug port.

If you instead want debug messages from real hardware, define
`TKEY_DEBUG` and the debug functions will use the debug HID device for
output.

If you use `TKEY_DEBUG` you *must* have something listening on the
corresponding HID device. It's usually the last HID device created. On
Linux, for instance, this means the last reported hidraw in `dmesg` is
the one you should do `cat /dev/hidrawX` on.

The emulator can output some memory access (and other) logs. You can
add `-d guest_errors` to the qemu command line to make QEMU send these
to stderr.

You can also use the QEMU monitor for debugging, for example, `info
registers`, or run QEMU with `-d in_asm` or `-d trace:riscv_trap` for
tracing.

#### GDB

If you run QEMU with `-s` it provides the GDB protocol on
`localhost:1234` (default).

If you run with `-S` QEMU doesn't start the firmware automatically. If
you also specified `-s` and attach a GDB you can control the start
entirely from GDB.

Your OS package system might include a GDB with RV32IMC support,
perhaps under a name like `riscv32-elf-gdb`. Ubuntu, however, does
not. Instead you can use GDB from

https://github.com/riscv-collab/riscv-gnu-toolchain/releases/

Before doing anything else, start qemu. Take note of what pty the
serial port ends up on. Look for:

```
char device redirected to /dev/pts/12 (label chrid)"
```

You can use this port later for communicating with your device app.

If you're debugging an ordinary device app, you have to first load the
device app, typically by using `tkey-runapp` from
[tkey-devtools](https://github.com/tillitis/tkey-devtools) using the
pty QEMU mentioned when starting:

```
tkey-runapp --port /dev/pts/12  app.bin
```

Then attach GDB to it:

```
riscv32-elf-gdb app.elf \
-ex "set architecture riscv:rv32" \
-ex "target remote :1234"
```

Attaching GDB works with both firmware and apps. For firmware
debugging you obviously don't need to use `tkey-runapp`.

**Note well**:

- You load the raw binary `app.bin` with `tkey-runapp` but you feed
  the ELF format binary `app.elf` to GDB.

- Remember to compile your programs with `-g` in `CFLAGS` to include
  debug symbols in the ELF file.

## Client applications

Typically you start a client application by connecting to the TKey:

```go
package main

import (
    "fmt"

	"github.com/tillitis/tkeyclient"
)

func main() {
	tk := tkeyclient.New()
	err := tk.Connect("/dev/ttyACM0")
	if err != nil {
		panic("Couldn't connect to TKey!")
	}
```

Then you can start using it by asking it to identify itself and make
sure it's in firmware mode:

```go
	nameVer, err := tk.GetNameVersion()
	if err != nil {
		...
	}

	fmt.Printf("Firmware name0:'%s' name1:'%s' version:%d\n",
		nameVer.Name0, nameVer.Name1, nameVer.Version)

```

Then you can load and start an app on the TKey:

```go
	err = tk.LoadAppFromFile("blink.bin", []byte{})
	if err != nil {
		panic(err)
	}
```

After this, the client program and the device program talk their own
protocol. You are encouraged to use the same framing protocol used
for the firmware while still replying negatively to a frame meant for
the firmware.

Please reply NOK to everything where the `domain` in the framing
header is `DST_FW` to allow a probing client app to know that it's
speaking to a running device app.

See the `tkeyclient` [Go
doc](https://pkg.go.dev/github.com/tillitis/tkeyclient)
for details on how to call the functions.
