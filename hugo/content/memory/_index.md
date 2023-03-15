---
title: Memory Map
weight: 3
---

# Memory map

## Addressing

```
31st bit                              0th bit
v                                     v
0000 0000 0000 0000 0000 0000 0000 0000
```

- Bits [31 .. 30] (2 bits): Top level prefix (described below)
- Bits [29 .. 24] (6 bits): Core select. We want to support at least 16 cores
- Bits [23 ..  0] (24 bits): Memory/in-core address.


## Assigned top level prefixes

| *name*   | *prefix* | *address length*                     |
|----------|----------|--------------------------------------|
| ROM      | 0b00     | 30 bit address                       |
| RAM      | 0b01     | 30 bit address                       |
| reserved | 0b10     |                                      |
| MMIO     | 0b11     | 6 bits for core select, 24 bits rest |

## Address prefixes

| *name*      | *prefix* |
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

MMIO begins at `0xc000_0000` but please use the constants in [tk1_mem.h](https://github.com/tillitis/tillitis-key1-apps/blob/main/apps/include/tk1_mem.h)

*Nota bene*: MMIO accesses should be 32 bit wide, e.g use `lw` and
`sw`. Exceptions are `FW_RAM` and `QEMU_DEBUG`.

| *name*            | *fw*  | *app*     | *size* | *type*   | *content* | *description*                                                           |
|-------------------|-------|-----------|--------|----------|-----------|-------------------------------------------------------------------------|
| `TRNG_STATUS`     | r     | r         |        |          |           | TRNG_STATUS_READY_BIT is 1 when an entropy word is available.           |
| `TRNG_ENTROPY`    | r     | r         | 4B     | u32      |           | Entropy word. Reading a word will clear status.                         |
| `TIMER_CTRL`      | r/w   | r/w       |        |          |           | If TIMER_STATUS_RUNNING_BIT in TIMER_STATUS is 0, setting               |
|                   |       |           |        |          |           | TIMER_CTRL_START_BIT here starts the timer.                             |
|                   |       |           |        |          |           | If TIMER_STATUS_RUNNING_BIT in TIMER_STATUS is 1, setting               |
|                   |       |           |        |          |           | TIMER_CTRL_STOP_BIT here stops the timer.                               |
| `TIMER_STATUS`    | r     | r         |        |          |           | TIMER_STATUS_RUNNING_BIT is 1 when the timer is running.                |
| `TIMER_PRESCALER` | r/w   | r/w       | 4B     |          |           | Prescaler init value. Write blocked when running.                       |
| `TIMER_TIMER`     | r/w   | r/w       | 4B     |          |           | Timer init or current value while running. Write blocked when running.  |
| `UDS_FIRST`       | r[^3] | invisible | 4B     | u8[32]   |           | First word of Unique Device Secret key.                                 |
| `UDS_LAST`        |       | invisible |        |          |           | The last word of the UDS                                                |
| `UART_BITRATE`    | r/w   |           |        |          |           | TBD                                                                     |
| `UART_DATABITS`   | r/w   |           |        |          |           | TBD                                                                     |
| `UART_STOPBITS`   | r/w   |           |        |          |           | TBD                                                                     |
| `UART_RX_STATUS`  | r     | r         | 1B     | u8       |           | Non-zero when there is data to read                                     |
| `UART_RX_DATA`    | r     | r         | 1B     | u8       |           | Data to read. Only LSB contains data                                    |
| `UART_RX_BYTES`   | r     | r         | 4B     | u32      |           | Number of bytes received from the host and not yet read by SW, FW.      |
| `UART_TX_STATUS`  | r     | r         | 1B     | u8       |           | Non-zero when it's OK to write data                                     |
| `UART_TX_DATA`    | w     | w         | 1B     | u8       |           | Data to send. Only LSB contains data                                    |
| `TOUCH_STATUS`    | r/w   | r/w       |        |          |           | TOUCH_STATUS_EVENT_BIT is 1 when touched. After detecting a touch       |
|                   |       |           |        |          |           | event (reading a 1), write anything here to acknowledge it.             |
| `FW_RAM`          | r/w   | invisible | 2 kiB  | u8[2048] |           | Firmware-only RAM.                                                      |
| `UDI`             | r     | invisible | 8B     | u64      |           | Unique Device ID (UDI).                                                 |
| `QEMU_DEBUG`      | w     | w         |        | u8       |           | Debug console (only in QEMU)                                            |
| `NAME0`           | r     | r         | 4B     | char[4]  | "tk1 "    | ID of core/stick                                                        |
| `NAME1`           | r     | r         | 4B     | char[4]  | "mkdf"    | ID of core/stick                                                        |
| `VERSION`         | r     | r         | 4B     | u32      | 1         | Current version.                                                        |
| `SWITCH_APP`      | r/w   | r         | 1B     | u8       |           | Write anything here to trigger the switch to application mode. Reading  |
|                   |       |           |        |          |           | returns 0 if device is in firmware mode, 0xffffffff if in app mode.     |
| `LED`             | r/w   | r/w       | 1B     | u8       |           | Control of the color LEDs in RBG LED on the board.                      |
|                   |       |           |        |          |           | Bit 0 is Blue, bit 1 is Green, and bit 2 is Red LED.                    |
| `GPIO`            | r/w   | r/w       | 1B     | u8       |           | Bits 0 and 1 contain the input level of GPIO 1 and 2.                   |
|                   |       |           |        | u8       |           | Bits 3 and 4 store the output level of GPIO 3 and 4.                    |
| `APP_ADDR`        | r/w   | r         | 4B     | u32      |           | Firmware stores app load address here, so app can read its own location |
| `APP_SIZE`        | r/w   | r         | 4B     | u32      |           | Firmware stores app app size here, so app can read its own size         |
| `BLAKE2S`         | r/w   | r         | 4B     | u32      |           | Function pointer to a BLAKE2S function in the firmware                  |
| `CDI_FIRST`       | r/w   | r         | 32B    | u8[32]   |           | Compound Device Identifier (CDI). UDS+measurement...                    |
| `CDI_LAST`        |       | r         |        |          |           | Last word of CDI                                                        |
| `RAM_ASLR`        | w     | invisible | 4B     | u32      |           | Address Space Randomization seed value for the RAM                      |
| `RAM_SCRAMBLE`    | w     | invisible | 4B     | u32      |           | Data scrambling seed value for the RAM                                  |

[^3]: The UDS can only be read *once* per power-cycle.
