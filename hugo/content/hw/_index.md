---
title: Hardware
weight: 3
---

# Hardware cores

## CPU

32 bit RISC-V running at 18 MHz. Architecture: RV32IC_Zmmul
(`-march=rv32iczmmul`).

No interrupts are used.

## FW ROM

The ROM memory containing the firmware. After reset the CPU will
read from the ROM to load, measure and start applications.

## RAM

128 kiB RAM. The memory is cleared by firmware before a TKey program
is loaded.

## Timer

A general purpose 32 bit timer. The timer will count down every cycle
from the initial value to one. In order to handle long time sequences
(minutes, hours, days) there is also a 32 bit prescaler. Since the CPU
is running at 18 MHz setting the prescaler to 18_000_000 means the
timer will tick every second.

## UART

A standard UART interface for receiving bytes from and send bytes
to the host via the interface microcontroller on the TKey.

The UART default speed is 62500 bps, but can be adjusted by the
program. (Note that the client must set the same bitrate too.)

The UART contain a 512 bit Rx-FIFO with status (data available).

#### ROSC

The ROSC is a ring oscillator based internal entropy source, or
True Random Number Generator (TRNG).

Note: The ROSC generates entropy with a fairly good quality. However
for security related use cases, for example keys, the ROSC should not
be used directly. Instead use it to create a seed for a Digital Random
Bit Generator (DRBG), also known as a Cryptographically Safe Pseudo
Random Number Generator (CSPRNG).

Examples of such generators are Hash_DRGG, CTR_DRBG, HKDF.

#### Touch sensor

The core provides a stable interface to the touch sensor on the
TKey device. Using the core, the firmware and applications can
get information about touch events and manage detection of
events.

#### TKey

The TKey core contains several functions, and acts as
main HW interface between firmware and applications. The core
includes:

- Read access to the 64 bit FPGA design name, expressed as ASCII chars.
- Read access to the 32 bit FPGA design version, expressed as an integer

- Control and status access for the RGB LED on TKey board
  - Setting bit 0 high turns on the Blue LED.
  - Setting bit 1 high turns on the Green LED.
  - Setting bit 2 high turns on the Red LED.

- Control and status access for the 4 GPIOs on the TKey board
  - GPIO 1 and 2 are inputs and provide read access to the
    current sampled value digital values on the pins.

  - GPIO 3 and 4 are outputs. The digital value written to
    the bits will be presented on the pins.

- Application read access to information about the loaded
  application. The information is written by the firmware.
  - Start address
  - Size of address

- Application read access to the CDI generated and written
  by the firmware when the application is loaded.

- Application-Firmware execution mode control. Can be read
  by the application and written by firmware. When written
  to by the firmware, the hardware will switch to application
  mode and start executing the application.
