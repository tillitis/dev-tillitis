---
title: Hardware
weight: 3
---

# Hardware Functions

VB: I introduce the less common abbreviations.

The TKey hardware contains the following hardware functions:
* CPU
* Execution monitor
* Firmware ROM
* RAM
* Address Space Layout Randomization (ASLR)
* RAM Scrambler
* Timer
* Universal Asynchronous Receiver/Transmitter (UART)
* True Random Number Generator (TRNG)
* Touch sensor

## CPU

The CPU is a 32-bit RISC-V running at 18 MHz with an RV32IC_Zmmul
(`-march=rv32iczmmul`) architecture.

VB: What does this mean?

No interrupts are used.

If the CPU encounters an illegal instruction, it halts and flashes the red LED.

## Execution monitor

VB: "Watch" can be misunderstood as a timekeeper instead of a guard.

VB: Where and why do you set start and end addresses? Explain.

The execution monitor can limit the program counter from reaching a certain memory utilization. You set a start and end address.

If the program counter is about to enter a forbidden memory area, the execution monitor feeds the execution monitor an illegal instruction which halts the CPU.

TODO Add how to set up.

## Firmware ROM

The ROM contains the firmware. After a reset, the CPU reads
from the ROM when loading, measuring, and starting applications.

## RAM

The RAM is 128 kB. 
The firmware clears the RAM before loading a TKey program into the RAM.

## ASLR

TODO

## RAM Scrambler

TODO

## Timer

VB: What is the initial value? Worth mentioning?

The general purpose 32-bit timer counts down every cycle
from the initial value to one. In order to handle long time sequences
(minutes, hours, days) there is also a 32-bit prescaler. 
If the prescaler is set to 18_000_000, the timer ticks every second
because the CPU is running at 18 MHz.

## UART

A standard UART interface for receiving bytes from and send bytes
to a TKey program via the interface microcontroller on the TKey.

The UART default speed is 62500 bps, but can be adjusted by any
TKey program. (Note that the client program must use the same 
bitrate too.)

The UART contains a 512-bit Rx-FIFO with status (data available).

VB: I don't get this. Can you rephrase the sentence?

Poll `UART_RX_STATUS` until it is non-zero, then a byte is available
in the LSB of the `UART_RX_DATA` word.

VB: I don't get this. Can you rephrase this sentence too?

Poll `UART_TX_STATUS` until it is non-zero, then you can write a byte
in the LSB of the `UART_TX_DATA` word.

#### TRNG

VB: I introduce the abbreviation in the beginning of the document.

The TRNG ring oscillator is based on an internal entropy source. 

VB: Is the programmer encouraged to do this or is it the TRNG
function which does the polling?

Poll `TRNG_STATUS` until non-zero, then read a word of
entropy from `TRNG_ENTROPY`.

The TRNG generates randomness with a fairly good quality. However for
security related use cases, for example generating keys, the TRNG should 
not be used directly. Instead use it to create a seed for a Digital Random
Bit Generator (DRBG), also known as a Cryptographically Safe Pseudo
Random Number Generator (CSPRNG).

Examples of such generators are Hash\_DRGG, CTR\_DRBG, HKDF.

#### Touch Sensor

The hardware core provides an interface to the touch sensor on the
TKey device. The touch sensor can share touch events with the
core, firmware, and applications.

VB: Explain

`TOUCH_STATUS`

#### TKey

The TKey core contains several functions, and acts as the main hardware
interface between the firmware and TKey programs. The core has:

- Read access to the 64 bit FPGA design name, expressed as ASCII
  chars: `NAME0` & `NAME1`.
  
- Read access to the 32 bit FPGA design version, expressed as an
  integer: `VERSION`.

- Control of and status access for the RGB LED on the TKey, called 
  `LED` in the memory map.
  - Setting bit 0 high turns on the Blue LED.
  - Setting bit 1 high turns on the Green LED.
  - Setting bit 2 high turns on the Red LED.

- Control of and status access for the 4 General-purpose Input/Output 
  (GPIOs) on the TKey, called `GPIO` in the memory map.
  - GPIO 1 and 2 are inputs and provide read access to the
    current sampled value digital values on the pins.

  - GPIO 3 and 4 are outputs. The digital value written to
    the bits is presented on the pins.

- Application read access to information about the loaded
  application. The information is written by the firmware.
  - Start address, called `APP_ADDR` in the memory map.
  - Size of program, called `APP_SIZE` in the memory map.

- Application read access to the Compound Device Identifier generated
  and written by the firmware when the application is loaded, called
  `CDI_FIRST` and `CDI_LAST` in the memory map.

- Application-Firmware execution mode control. Can be read by programs
  and written by firmware. When written to by the firmware, the
  hardware will switch to application mode, called `SWITCH_APP` 
  in the memory map.
