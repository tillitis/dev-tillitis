---
title: Hardware
weight: 3
---

# Hardware cores

## CPU

TKey has a 32-bit RISC-V running at 18 MHz with an RV32IC_Zmmul
(`-march=rv32iczmmul`) architecture.

VB: What does this mean?

No interrupts are used.

VB: Illegal instruction from and to?

Any illegal instruction from XXXXX to YYYYY halts the CPU and makes the red TKey LED flash.

## Execution monitor

VB: First introduce what the execution monitor is. For example: "The TKey firmware contains an execution monitor that can prevent..."

VB: The relation between the objects in the first sentence is not clear.

VB: Where and why do you set start and end addresses?

The execution monitor can be used to set up a watch that the program counter doesn't reach certain memory. You set a start and end address.

If the program counter is about to enter a forbidden memory area, the execution monitor feeds the monitor an illegal instruction which halts the CPU.

TODO Add how to set up.

## FW ROM

The ROM memory containing the firmware. After reset the CPU will
read from the ROM to load, measure and start applications.

## RAM

128 kiB RAM. The memory is cleared by firmware before a TKey program
is loaded.

## ASLR

TODO

## RAM Scrambler

TODO

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

Poll `UART_RX_STATUS` until it is non-zero, then a byte is available
in the LSB of the `UART_RX_DATA` word.

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

The core provides a stable interface to the touch sensor on the
TKey device. Using the core, the firmware and applications can
get information about touch events and manage detection of
events.

`TOUCH_STATUS`

#### TKey

The TKey core contains several functions, and acts as main hardware
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

- Control and status access for the 4 GPIOs on the TKey board, `GPIO`
  in memory map.
  - GPIO 1 and 2 are inputs and provide read access to the
    current sampled value digital values on the pins.

  - GPIO 3 and 4 are outputs. The digital value written to
    the bits will be presented on the pins.

- Application read access to information about the loaded
  application. The information is written by the firmware.
  - Start address, `APP_ADDR`.
  - Size of program, `APP_SIZE`.

- Application read access to the Compound Device Identifier generated
  and written by the firmware when the application is loaded,
  `CDI_FIRST` to `CDI_LAST`.

- Application-Firmware execution mode control. Can be read by programs
  and written by firmware. When written to by the firmware, the
  hardware will switch to application mode, `SWITCH_APP`.
