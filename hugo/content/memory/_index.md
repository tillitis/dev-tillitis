---
title: Memory Map
weight: 3
---

# Memory Map

VB: Introduce what kind of memory map this is like: "This is the memory map for XXXX with prefixes for ..." so that the reader gets some context.

## Addressing
VB: Give some background info/context.

This is an example of a ...............

```
31st bit                              0th bit
v                                     v
0000 0000 0000 0000 0000 0000 0000 0000
```
VB: Introduce the list in some way. Maybe something like: "This happens if the following bits are set in X:"

- Bits [31 .. 30] (2 bits): Top level prefix (described below)
- Bits [29 .. 24] (6 bits): Core select. We want to support at least 16 cores
- Bits [23 ..  0] (24 bits): Memory/in-core address.


## Assigned Top Level Prefixes

VB: If these are the prefixes that someone could use in their program then I wonder if a title like "Available Top Level Prefixes" would reflect that better. 

VB: Give some context and intro to the table.

| *Name*   | *Prefix* | *Address length*                     |
|----------|----------|--------------------------------------|
| ROM      | 0b00     | 30 bit address                       |
| RAM      | 0b01     | 30 bit address                       |
| reserved | 0b10     |                                      |
| MMIO     | 0b11     | 6 bits for core select, 24 bits rest |

## Address Prefixes

VB: Give some context and intro to the table.

| *Name*      | *Prefix* |
|-------------|----------|
| ROM         | 0x00     |
| RAM         | 0x40     |
| MMIO        | 0xc0     |
| MMIO TRNG   | 0xc0     |
| MMIO TIMER  | 0xc1     |
| MMIO UDS    | 0xc2     |
| MMIO UART   | 0xc3     |
| MMIO TOUCH  | 0xc4     |
| MMIO FW_RAM | 0xd0     |
| MMIO TK1    | 0xff     |

TODO fill in rest

## MMIO

VB: From my perspective, some context would help.

VB: Consider if MMIO is an abbreviation that should be introduced.

VB: Add info about when the "contants" should be used.

MMIO begins at `0xc000_0000` but please use the constants in [tk1_mem.h](https://github.com/tillitis/tillitis-key1-apps/blob/main/apps/include/tk1_mem.h)

VB: When should MMIO access be 32 bit.

VB: When should "lw" and "sw" be used?

*Note*: MMIO accesses should be 32 bit wide. For example, use `lw` and
`sw`. Exceptions are `FW_RAM` and `QEMU_DEBUG`.

In the following table, *r* equals read and *r/w* equals read and write.

| *Name*            | *Firmware*  | *Program*     | *Size* | *Type*   | *Content* | *Description*                                                 |
|-------------------|-------|-----------|--------|----------|-----------|-------------------------------------------------------------------------|
| `TRNG_STATUS`     | r     | r         |        |          |           | TRNG_STATUS_READY_BIT is 1 when an entropy word is available.           |
| `TRNG_ENTROPY`    | r     | r         | 4B     | u32      |           | Entropy word. Reading a word clears the status. VB: Status of what?           |
| `TIMER_CTRL`      | r/w   | r/w       |        |          |           | If TIMER_STATUS_RUNNING_BIT in TIMER_STATUS is 0, setting VB: missing info?   |
| VB: Missing info? |       |           |        |          |           | If TIMER_CTRL_START_BIT is set here starts the timer. VB: Missing info?       |
| VB: Missing info? |       |           |        |          |           | If TIMER_STATUS_RUNNING_BIT in TIMER_STATUS is 1, setting VB: Missing info?   |
| VB: Missing info? |       |           |        |          |           | TIMER_CTRL_STOP_BIT here stops the timer.                               |
| `TIMER_STATUS`    | r     | r         |        |          |           | TIMER_STATUS_RUNNING_BIT is 1 when the timer is running.                |
| `TIMER_PRESCALER` | r/w   | r/w       | 4B     |          |           | Prescaler init value. Write blocked when running.                       |
| `TIMER_TIMER`     | r/w   | r/w       | 4B     |          |           | Timer init or current value while running. Write blocked when running.  |
| `UDS_FIRST`       | r[^3] | invisible | 4B     | u8[32]   |           | First word of Unique Device Secret key.                                 |
| `UDS_LAST`        |       | invisible |        |          |           | The last word of the UDS.                                                |
| `UART_BITRATE`    | r/w   |           |        |          |           | TBD                                                                     |
| `UART_DATABITS`   | r/w   |           |        |          |           | TBD                                                                     |
| `UART_STOPBITS`   | r/w   |           |        |          |           | TBD                                                                     |
| `UART_RX_STATUS`  | r     | r         | 1B     | u8       |           | Non-zero when there is data to read.                                    |
| `UART_RX_DATA`    | r     | r         | 1B     | u8       |           | Data to read. Only the LSB contains data.                               |
| `UART_RX_BYTES`   | r     | r         | 4B     | u32      |           | Number of bytes received from the host and not yet read by the SW or FW.|
| `UART_TX_STATUS`  | r     | r         | 1B     | u8       |           | Non-zero when it's OK to write data.                                    |
| `UART_TX_DATA`    | w     | w         | 1B     | u8       |           | Data to send. Only the LSB contains data.                               |
| `TOUCH_STATUS`    | r/w   | r/w       |        |          |           | TOUCH_STATUS_EVENT_BIT is 1 when touched. After detecting a touch VB: Missing info? |
|                   |       |           |        |          |           | event (reading a 1), write anything here to acknowledge it. VB: Change it to the object.      |
| `FW_RAM`          | r/w   | invisible | 2 kiB  | u8[2048] |           | Firmware-only RAM.                                                      |
| `UDI`             | r     | invisible | 8B     | u64      |           | Unique Device ID (UDI).                                                 |
| `QEMU_DEBUG`      | w     | w         |        | u8       |           | Debug console (only in QEMU)                                            |
| `NAME0`           | r     | r         | 4B     | char[4]  | "tk1 "    | ID of core/stick                                                        |
| `NAME1`           | r     | r         | 4B     | char[4]  | "mkdf"    | ID of core/stick                                                        |
| `VERSION`         | r     | r         | 4B     | u32      | 1         | Current version. VB: Version of the program sw?                         |
| `SWITCH_APP`      | r/w   | r         | 1B     | u8       |           | Write anything here to trigger the switch to application mode. Reading VB: Missing info? |
|                   |       |           |        |          |           | Returns 0 if device is in firmware mode and 0xffffffff if in app mode.  |
| `LED`             | r/w   | r/w       | 1B     | u8       |           | Controls the RGB color of the status indicator LED on TKey.             |
|VB: Missing info?  |       |           |        |          |           | Bit 0 shows blue, bit 1 shows green, and bit 2 shows red LED.           |
| `GPIO`            | r/w   | r/w       | 1B     | u8       |           | Bits 0 and 1 contain the input level of GPIO 1 and 2.                   |
|                   |       |           |        | u8       |           | Bits 3 and 4 store the output level of GPIO 3 and 4.                    |
| `APP_ADDR`        | r/w   | r         | 4B     | u32      |           | The firmware stores the program load address here, so that the program can read its own location. VB: "Location" as in USB port or similar? |
| `APP_SIZE`        | r/w   | r         | 4B     | u32      |           | The firmware stores the program size here, so that the program can read its own size.         |
| `BLAKE2S`         | r/w   | r         | 4B     | u32      |           | Function pointer to a BLAKE2S function in the firmware.                  |
| `CDI_FIRST`       | r/w   | r         | 32B    | u8[32]   |           | Compound Device Identifier (CDI). UDS+measurement...   VB: Missing info? |
| `CDI_LAST`        |       | r         |        |          |           | Last word of the CDI.                                                    |
| `RAM_ASLR`        | w     | invisible | 4B     | u32      |           | Address Space Randomization seed value for the RAM.                      |
| `RAM_SCRAMBLE`    | w     | invisible | 4B     | u32      |           | Data scrambling seed value for the RAM.                                  |

[^3]: The UDS is read *one time* when TKey starts.
