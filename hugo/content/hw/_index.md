---
title: Hardware
weight: 3
---

# TKey Hardware Design

The TKey hardware design consists of a FPGA device and some support
components mounted on the PCB. The FPGA design contain the TKey System
on Chip (SoC) where device applications are loaded and executed.
Communication between the TKey SoC and a client app (running on the
host computer) is handled by a small microcontroller (MCU) that
handles conversion between USB and UART.

The SoC, called application_fpga consists of a CPU and a number of
cores attached through the memory subsystem. The following is a
description of the CPU, the cores, and the functionality they
implement from a developer point of view.

## CPU

The CPU of the TKey is a 32-bit RISC-V running at 18 MHz. It supports
the RV32 instruction set with RV32IC_Zmmul (`-march=rv32iczmmul`)
extensions.

This means that the base RV23I as well as compressed instructions are
supported. The subset of the M(ath) extension that includes
multiplication, but not division is supported.

There is no support for interrupts.

Any illegal, unsupported instruction will halt the CPU. The halted CPU
is detected by the hardware, which will blink the RGB LED with red to
indicate the error state.

## Execution Monitor

The execution monitor can be used by TKey apps to prevent the CPU from
trying to execute instructions from a defined memory area.

When the execution monitor detects that the CPU is requesting
instructions from the area, the monitor will force the CPU to read an
illegal instruction. This will halt the CPU until reset.

The execution monitor is set up by writing the start and end address
to the registers in the TK1 core. When the addresses has been reset,
the monitor is enabled by writing to the monitor control register.
When enabled the monitor can not be disabled and the addresses can't
be altered.

Note that the monitor also protects the FW (firmware) RAM. This area
is always protected.


## Firmware ROM

The ROM (Read-Only Memory) contains the firmware. After a reset, the
CPU will start executing the code at the beginning of the ROM, thus
running the firmware. The firmware is then responsible for receiving,
measuring and starting a device application. The ROM image is part of
the FPGA bitstream.

## RAM

The RAM is 128 kiB. The firmware clears the RAM before loading a the
TKey device app into RAM.

# Address Space Layout Randomization (ASLR)

The TKey hardware includes a simple form of RAM memory protection. The
memory protection is based on two separate mechanisms:

1. Address Space Layout Randomisation (ASLR)
2. Address dependent data scrambling

The ASLR is implemented by XORing the CPU address with the contents of
the `ADDR_RAM_ASLR` register in the TK1 core. The result is used as
the RAM address. The ASLR is set up by the firmware as part of loading
the TKey device app. The ASLR will be transparent to the app, and
developers does not have to do anything to use it.

For more information about the ASLR, please see the Tillitis Key
[system
description](https://github.com/tillitis/tillitis-key1/blob/main/doc/system_description/system_description.md)
(in the tillitis-key1 repository).

## RAM Scrambler

The data scrambling is implemented by XORing the data written to the
RAM with the contents of the `ADDR_RAM_SCRAMBLE` register in the TK1
core as well as XORing with the CPU address. This means that the same
data written to two different addresses will be scrambled differently.
The same pair or XOR operations is also performed on the data read out
from the RAM.

The data scrambling is set up by the firmware as part of loading the
TKey device app. The scrambling will be transparent to the app, and
developers does not have to do anything to use it.


## Timer

A general purpose 32-bit timer. The timer will count down every cycle,
from the initial value down to one. In order to handle long time
sequences (minutes, hours, days) there is also a 32-bit prescaler. If
the prescaler is set to 18_000_000, the timer ticks every second
because the CPU is running at 18 MHz.

## UART

A standard UART (Universal Asynchronous Receiver/Transmitter)
interface for receiving bytes from and send bytes to a TKey device app
via the interface microcontroller on the TKey. The UART default speed
is 62500 bps, but can be adjusted by the device app. (Note that the
client app must set the same bitrate too.)

The UART contains a 512-bit Rx-FIFO with status (data available).

The device app can read from the UART by polling `UART_RX_STATUS`
until it is non-zero. The received byte is then available in the LSB
(Least Significant Byte) of the `UART_RX_DATA` word.

Writing is done by polling `UART_TX_STATUS` until it is non-zero. The
byte to transmit can then be written to the LSB of the `UART_TX_DATA`
word.

## True Random Number Generator (TRNG)

The True Random Number Generator (TRNG) ring oscillator based internal
entropy source.

The TRNG generates randomness with a fairly good quality. However for
security related use cases, for example generating keys, the TRNG
should not be used directly. Instead use it to create a seed for a
Digital Random Bit Generator (DRBG), also known as a Cryptographically
Safe Pseudo Random Number Generator (CSPRNG). Examples of such
generators are Hash\_DRGG, CTR\_DRBG, HKDF.

Getting a word of entropy is done by polling `TRNG_STATUS` until
non-zero, then reading the word from `TRNG_ENTROPY`.

## Touch Sensor

The touch_sense core provides an interface to the touch sensor on the
TKey device. Using the core, the firmware as well as applications can
get information about touch events and manage detection of events.

It is recommended to start handling touch events by acknowledging any
stray event before signalling to the user that a touch event is
expected, and then start waiting for the event.


## TK1

The TKey core contains several functions, and acts as the main
hardware interface between firmware and device applications. The core
has:

- Read access to the 64 bit FPGA design name, expressed as ASCII
  chars: `NAME0` & `NAME1`.

- Read access to the 32 bit FPGA design version, expressed as an
  integer: `VERSION`.

- Control of and status access for the RGB LED on the TKey, called
  `LED` in the memory map.

  - Setting bit 0 high turns on the Blue LED.
  - Setting bit 1 high turns on the Green LED.
  - Setting bit 2 high turns on the Red LED.

- Control of and status access for the 4 GPIO (General-purpose
  Input/Output) pins on the TKey. Called `GPIO` in the memory map.

  - GPIO 1 and 2 are inputs and provide read access to the
    current sampled value digital values on the pins.

  - GPIO 3 and 4 are outputs. The digital value written to
    the bits is presented on the pins.

- Application read access to information about the loaded application.
  The information is written by the firmware.

  - Start address, called `APP_ADDR` in the memory map.
  - Size of device app, called `APP_SIZE` in the memory map.

- Application read access to the Compound Device Identifier (CDI)
  generated and written by the firmware when the application is
  loaded. Called `CDI_FIRST` to `CDI_LAST` in the memor map.

- Application-Firmware execution mode control. Can be written to by
  the firmware, and read by device app. When the firmware writes to
  it, the hardware will switch to application mode. Called
  `SWITCH_APP` in the memory map.
