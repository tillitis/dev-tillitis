---
title: Hardware
weight: 3
---

# TKey Hardware Design

The TKey hardware design consists of an FPGA device and some support
components mounted on the PCB. The FPGA design contain the TKey System
on Chip (SoC) where device applications are loaded and executed.
Communication between the TKey SoC and a client app (running on the
host computer) is handled by a small microcontroller (MCU) that
handles conversion between USB and UART.

The SoC, called `application_fpga` in [the hardware
repo](https://github.com/tillitis/tillitis-key1/) consists of a CPU
and a number of cores attached through the memory subsystem. The
following is a description of the CPU, the cores, and the
functionality they implement from a developer point of view.

## CPU

The CPU of the TKey is a modified version of
[PicoRV32](https://github.com/YosysHQ/picorv32), 32-bit RISC-V running
at 18 MHz. Modifications includes a fast 32x32 multiplier implemented
using the multiplier blocks in the iCE40 DSPs as well as a HW trap
function.

The supported instruction set supported by the CPU is a subset of
RV32I. Specifically it includes compressed instructions, but excludes
instructions for:

- Counters
- System
- Synch
- CSR access
- Change level
- Trap redirect
- Interrupt
- MMU

The instruction set implemented by the CPU also includes
multiplication instructions from the RV32IC\_Zmmul
(`-march=rv32iczmmul`) extension. Division is not supported.

Any illegal, unsupported instruction will halt the CPU. The halted CPU
is detected by the hardware, which will blink the RGB LED with red to
indicate the error state. There is no way for the CPU to exit the
trap state besides a power cycle of the device.

Note that the CPU has no support for interrupts. No instructions,
ports or logic.

## Security Monitor

The security monitor can be used by TKey apps to prevent the CPU from
trying to execute instructions from a memory area. When the monitor
detects that the CPU is requesting instructions from the area, the
monitor will force the CPU to read an illegal instruction. This will
halt the CPU until it is reset. This will also be indicated by a
red flashing status LED.

The monitor is set up by writing the start address to
`TK1_MMIO_TK1_CPU_MON_FIRST` and end addresses to
`TK1_MMIO_TK1_CPU_MON_LAST`, then starting the monitor by writing to
`TK1_MMIO_TK1_CPU_MON_CTRL`. After having been enabled the monitor
cannot be disabled and the addresses cannot be altered.

The monitor also protects the special firmware RAM, FW\_RAM. This area
is always protected.

## Firmware ROM

The ROM (Read-Only Memory) contains the firmware. After a reset, the
CPU will start executing the code at the beginning of the ROM, thus
running the firmware. The firmware is then responsible for receiving,
measuring and starting a device application. The ROM image is part of
the FPGA bitstream.

## RAM

The RAM begins at 0x4000\_0000 and ends at 0x4002\_0000 (128 kiB).

The firmware clears and fills the RAM with randomized words on
power up.

### Address Randomisation

The TKey hardware includes a simple form of RAM memory protection from
external threats. This might mitigate some of the problems of a warm
boot attack potentially reading out all the RAM contents.

The memory protection is based on two separate mechanisms:

1. Address randomisation
2. Address dependent data scrambling

The address randomisation is implemented by XORing the CPU address
with the contents of `TK1_MMIO_TK1_RAM_ADDR_RAND`. The result is used
as the RAM address. This is set up by the firmware as part of loading
the TKey device app. The addresses will be transparent to the device
app and developers don't have to do anything to use it.

For more information about this, please see the Tillitis Key
`doc/system description.md` file of earlier releases (in the
tillitis-key1 repository).

Note that this is not randomising offsets to the stack or well-known
functions. This is address randomisation as seen from *outside* of the
running process, meant to slow down a hardware attack.

### RAM Scrambler

The data scrambling is implemented by XORing the data written to the
RAM with the contents of `TK1_MMIO_TK1_RAM_SCRAMBLE` as well as XORing
with the CPU address. This means that the same data written to two
different addresses will be scrambled differently. The same pair or
XOR operations is also performed on the data read out from the RAM.

The data scrambling is set up by the firmware as part of loading the
TKey device app. The scrambling will be transparent to the app and
developers don't have to do anything to use it.

## Timer

A general purpose 32-bit timer. The timer will count down every cycle,
from the initial value down to one. In order to handle long time
sequences (minutes, hours, days) there is also a 32-bit prescaler.

Typical use:

1. Set the prescaler by writing to `TK1_MMIO_TIMER_PRESCALER`. Default
   is 1. If you set the prescaler to 18\_000\_000, the timer ticks
   every second because the CPU is running at 18 MHz.
2. Set the initialization value of the timer by writing to
   `TK1_MMIO_TIMER_TIMER`.
3. Start the timer by setting bit 0 in `TK1_MMIO_TIMER_CTRL`.
4. Poll status by checking bit if 0 is 1 in `TK1_MMIO_TIMER_STATUS`.
   If it is 0, the timer has expired.

If you want to stop the timer, set bit 1 in `TK1_MMIO_TIMER_CTRL`.

## UART

A standard UART (Universal Asynchronous Receiver/Transmitter)
interface is used for sending and receiving bytes to a TKey device app
via the interface microcontroller on the TKey. The UART configuration is:
- baudrate: 62500 bps
- data bits: 8
- stop bit: 1
- parity: none

Note that the client app must set the same configuration. Configuring
the UART core's baudrate, bit rate and stop bits, in runtime, is
deprecated and should not be altered by an app. This is to ensure
compatibility and prevent mismatch between different TKeys. The
baudrate is set when building the bitstream.

The UART contains a 512-byte Rx-FIFO with status (data available).

The device app can read from the UART by polling
`TK1_MMIO_UART_RX_STATUS` until it is non-zero. The received byte is
then available in the LSB (Least Significant Byte) of the
`TK1_MMIO_UART_RX_DATA` word.

Writing is done by polling `TK1_MMIO_UART_TX_STATUS` until it is
non-zero. The byte to transmit can then be written to the LSB of the
`TK1_MMIO_UART_TX_DATA` word.

See `proto.c` in tkey-libs.

## True Random Number Generator (TRNG)

The True Random Number Generator (TRNG) ring oscillator based internal
entropy source.

The TRNG generates randomness with a fairly good quality. However for
security related use cases, for example generating keys, the TRNG
should not be used directly. Instead use it to create a seed for a
Digital Random Bit Generator (DRBG), also known as a Cryptographically
Safe Pseudo Random Number Generator (CSPRNG). Examples of such
generators are Hash\_DRGG, CTR\_DRBG, HKDF.

Getting a word of entropy is done by polling `TK1_MMIO_TRNG_STATUS`
until non-zero, then reading the word from `TK1_MMIO_TRNG_ENTROPY`.

## Touch Sensor

The touch\_sense core provides an interface to the touch sensor on the
TKey device. Using the core, the firmware as well as applications can
get information about touch events and manage detection of events.

It is recommended to start handling touch events by acknowledging any
stray event before signalling to the user that a touch event is
expected, and then start waiting for the event.

1. Begin by writing to `TK1_MMIO_TOUCH_STATUS` to acknowledge any
   stray events.
2. Poll `TK1_MMIO_TOUCH_STATUS`. When non-zero the sensor has detected
   a touch.
3. Acknowledge the touch by writing to `TK1_MMIO_TOUCH_STATUS`.

See `touch_wait()` in tkey-libs.

## UDS

Only available in firmware mode. 8 words of Unique Device Secret. Part
of the FPGA design, changed when provisioning a TKey. It is available
between the words `TK1_MMIO_UDS_FIRST` and `TK1_MMIO_UDS_LAST`.

The UDS is only readable once per power cycle. The protection is done
per word, so make sure to only access UDS by words.

## TK1

The TKey core contains several functions, and acts as the main
hardware interface between firmware and device applications. The core
has:

- Read access to the 64 bit FPGA design name, expressed as ASCII
  chars: `TK1_MMIO_TK1_NAME0` & `TK1_MMIO_TK1_NAME1`.

- Read access to the 32 bit FPGA design version, expressed as an
  integer: `TK1_MMIO_TK1_VERSION`.

- Control of and status access for the RGB status LED on the TKey,
  available at `TK1_MMIO_TK1_LED`.

  - Setting bit 0 high turns on the Blue LED.
  - Setting bit 1 high turns on the Green LED.
  - Setting bit 2 high turns on the Red LED.

  See `led_get()` and `led_set()` and constants in tkey-libs.

- Control of and status access for the 4 GPIO (General-purpose
  Input/Output) pins on the TKey, available at `TK1_MMIO_TK1_GPIO`.

  - GPIO 1 and 2 are inputs and provide read access to the
    current sampled value digital values on the pins.

  - GPIO 3 and 4 are outputs. The digital value written to
    the bits is presented on the pins.

- Application read access to information about the loaded application.
  The information is written by the firmware.

  - Start address at `TK1_MMIO_TK1_APP_ADDR`.
  - Size of device app at `TK1_MMIO_TK1_APP_SIZE`.

- Application read access to 8 words of Compound Device Identifier
  (CDI) generated and written by the firmware when the application is
  loaded. They are available between `TK1_MMIO_TK1_CDI_FIRST` and
  `TK1_MMIO_TK1_CDI_LAST`.

- Application-Firmware execution mode control, available at
  `TK1_MMIO_TK1_SWITCH_APP`. Can be written to by the firmware, and
  read by device app. When the firmware writes to it, the hardware
  will switch to application mode, which closes off some of the memory
  app for reading and or writing. See the table in [the memory
  map](/memory).

  If read in application mode it returns `0xffffffff`.

- Two words of Unique Device Identifier (UDI) available between
  `TK1_MMIO_TK1_UDI_FIRST` and `TK1_MMIO_TK1_UDI_LAST`. Only available
  in firmware mode.

- An address to the built-in BLAKE2s hash function in the firmware on
  `TK1_MMIO_TK1_BLAKE2S`. Wrapped in tkey-libs so you can use it like
  an ordinary C function:

  ```c
  int blake2s(void *out, unsigned long outlen, const void *key,
	    unsigned long keylen, const void *in, unsigned long inlen,
	    blake2s_ctx *ctx)
  ```

## QEMU

There is a simple debug port available which is usable only in [the
TKey emulator](https://github.com/tillitis/qemu/tree/tk1).

Write a byte to `TK1_MMIO_QEMU_DEBUG` to get it printed in the qemu
console.

See `qemu_debug.h` in tkey-libs for helper functions.
