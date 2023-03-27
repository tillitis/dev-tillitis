---
title: Hardware
weight: 3
---

# TKey hardware design

The TKey hardware design consists of a FPGA device and some support
components mounted on the PCB. The FPGA design contain the TKey System
on Chip (SoC) that TKey host apps are loaded and executed.
Communication between the Tkey SoC and the client is handled by a
small microcontroller (MCU) that handles conversion between USB and
UART.

The SoC, called application_gpga consists of a CPU and a number of
cores attached to the through the memory sub system. The following is
a description of the CPU and the cores from a developer point of view.

## CPU

TKey has a 32-bit RISC-V running at 18 MHz. The CPU supports the RV32
instruction set with RV32IC_Zmmul (`-march=rv32iczmmul`) extensions.

This means that the base RV23I as well as compressed instructions are
supported. The subset of the M(ath) extension that includes
multiplication, but not division is supported.

There is no support for interrupts.

Illegal, unsupported instructions halts the CPU. CPU halting is
detected by the hardware and will blink the RGB LED with Red to
indicate the error state. LED red forever.

## Execution monitor

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


## FW ROM

The ROM memory contains the the firmware. After reset the CPU will
read from the ROM to load, measure and start applications. The ROM
image is part of the FPGA bitstream.

## RAM

128 kiB RAM. The memory is cleared by firmware before a TKey program
is loaded.

## ASLR

The TKey hardware includes a simple form of RAM memory protection. The
memory protection is based on two separate mechanisms:

1. Address Space Layout Randomisation (ASLR)
2. Address dependent data scrambling

The ASLR is implemented by XORing the CPU address with the contents of
the `ADDR_RAM_ASLR` register in the TK1 core. The result is used as
the RAM address. The ASLR is set up by the FW as part of loading the
Tkey device app. The ASLR will be transparent to the app, and
developers does not have to do anything to use it.

For more information about the ASLR, please see the Tillitis Key
[system
description](https://github.com/tillitis/tillitis-key1/blob/main/doc/system_description/system_description.md).


## RAM Scrambler

The data scrambling is implemented by XORing the data written to the
RAM with the contents of the `ADDR_RAM_SCRAMBLE` register in the TK1
core as well as XORing with the CPU address. This means that the same
data written to two different addresses will be scrambled differently.
The same pair or XOR operations is also performed on the data read out
from the RAM.

The data scrambling is set up by the FW as part of loading the Tkey
device app. The scrambling will be transparent to the app, and
developers does not have to do anything to use it.


## Timer

A general purpose 32 bit timer. The timer will count down every cycle
from the initial value to one. In order to handle long time sequences
(minutes, hours, days) there is also a 32 bit prescaler. Since the CPU
is running at 18 MHz setting the prescaler to 18_000_000 means the
timer will tick every second.

## UART

A standard UART interface for receiving bytes from and send bytes to
the host via the interface microcontroller on the TKey.

The UART default speed is 62500 bps, but can be adjusted by the
program. (Note that the client must set the same bitrate too.)

The UART contain a 512 bit Rx-FIFO with status (data available).

Poll `UART_RX_STATUS` until it is non-zero, then a byte is available
in the LSB (Least Significant Bit) of the `UART_RX_DATA` word.

Poll `UART_TX_STATUS` until it is non-zero, then you can write a byte
in the LSB of the `UART_TX_DATA` word.

#### TRNG

The True Random Number Generator (TRNG) ring oscillator based internal
entropy source. Poll `TRNG_STATUS` until non-zero, then read a word of
entropy from `TRNG_ENTROPY`.

The TRNG generates entropy with a fairly good quality. However for
security related use cases, for example keys, the TRNG should not be
used directly. Instead use it to create a seed for a Digital Random
Bit Generator (DRBG), also known as a Cryptographically Safe Pseudo
Random Number Generator (CSPRNG).

Examples of such generators are Hash\_DRGG, CTR\_DRBG, HKDF.

#### Touch sensor

The core provides a stable interface to the touch sensor on the TKey
device. Using the core, the firmware and applications can get
information about touch events and manage detection of events.

#### TK1

The TK1 core contains several functions, and acts as main hardware
interface between firmware and TKey programs. The core includes:

- Read access to the 64 bit FPGA design name, expressed as ASCII
  chars: `NAME0` & `NAME1`.
- Read access to the 32 bit FPGA design version, expressed as an
  integer: `VERSION`.

- Control and status access for the RGB LED on TKey board, `LED` in
  memory map.

  - Setting bit 0 high turns on the Blue LED.
  - Setting bit 1 high turns on the Green LED.
  - Setting bit 2 high turns on the Red LED.

- Control and status access for the 4 GPIO pins on the TKey board,
  `GPIO` in memory map.

  - GPIO 1 and 2 are inputs and provide read access to the
    current sampled value digital values on the pins.

  - GPIO 3 and 4 are outputs. The digital value written to
    the bits will be presented on the pins.

- Application read access to information about the loaded application.
  The information is written by the firmware.

  - Start address, `APP_ADDR`.
  - Size of program, `APP_SIZE`.

- Application read access to the Compound Device Identifier generated
  and written by the firmware when the application is loaded,
  `CDI_FIRST` to `CDI_LAST`.

- Application-Firmware execution mode control. Can be read by programs
  and written by firmware. When written to by the firmware, the
  hardware will switch to application mode, `SWITCH_APP`.
