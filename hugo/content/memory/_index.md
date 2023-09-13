---
title: Memory Map
weight: 3
---

# Memory Map

## Addressing

Layout of a 32-bit address:

```
31st bit                              0th bit
v                                     v
0000 0000 0000 0000 0000 0000 0000 0000
```

- Bits [31 .. 30] (2 bits): Top level prefix (described below)
- Bits [29 .. 24] (6 bits): Core select. There is space for 64 cores.
- Bits [23 ..  0] (24 bits): Actual memory or in-core address.


## Assigned Top Level Prefixes

The first 2 bits in a 32-bit address.

| *name*   | *prefix* | *address length*                     |
|----------|----------|--------------------------------------|
| ROM      | 0b00     | 30 bit address                       |
| RAM      | 0b01     | 30 bit address                       |
| reserved | 0b10     |                                      |
| MMIO     | 0b11     | 6 bits for core select, 24 bits rest |

## Address Prefixes

The first 8 bits in a 32-bit address.

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

## MMIO

MMIO begins at `0xc000_0000` but please use the constants in
[tk1_mem.h](https://github.com/tillitis/tkey-libs/blob/main/include/tk1_mem.h)
in the `tkey-libs` repository.

*Note*: MMIO accesses should be 32 bits wide. Use for example `lw` and
`sw` to load and store 32-bit words. Exceptions are `FW_RAM` and
`QEMU_DEBUG`.

In the following table, *r* equals read and *r/w* equals read and
write access.

| *Name*            | *FW*  | *App*     | *Size* | *Type*   | *Content* | *Description*                                                           |
|-------------------|-------|-----------|--------|----------|-----------|-------------------------------------------------------------------------|
| `TRNG_STATUS`     | r     | r         |        |          |           | `TRNG_STATUS_READY_BIT` is 1 when an entropy word is available.         |
| `TRNG_ENTROPY`    | r     | r         | 4B     | u32      |           | Entropy word. Reading a word will clear `TRNG_STATUS`.                  |
| `TIMER_CTRL`      | r/w   | r/w       |        |          |           | If `TIMER_STATUS_RUNNING_BIT` in `TIMER_STATUS` is 0, setting `TIMER_CTRL_START_BIT` here starts the timer. If `TIMER_STATUS_RUNNING_BIT` in `TIMER_STATUS` is 1, setting `TIMER_CTRL_STOP_BIT` here stops the timer. |
| `TIMER_STATUS`    | r     | r         |        |          |           | `TIMER_STATUS_RUNNING_BIT` is 1 when the timer is running.              |
| `TIMER_PRESCALER` | r/w   | r/w       | 4B     |          |           | Prescaler init value. Write blocked when running.                       |
| `TIMER_TIMER`     | r/w   | r/w       | 4B     |          |           | Timer init or current value while running. Write blocked when running.  |
| `UDS_FIRST`       | r[^3] | invisible | 32B     | u32[8]  |           | First word of Unique Device Secret key. Word access only. **Note:** Only readable once per power up. |
| `UDS_LAST`        |       | invisible |        |          |           | The last word of the UDS. Note: Only readable once per power up.        |
| `UART_BITRATE`    | r/w   | r/w       | 2B     | u16      |           | Default 288 (62 500 bps). The bitrate is set by writing the divisor, calculated by: divisor = 18E6 / bps. |
| `UART_DATABITS`   | r/w   | r/w       | 4 bits | u8       |           | Default 8.                                                              |
| `UART_STOPBITS`   | r/w   | r/w       | 2 bits | u8       |           | Default 1.                                                              |
| `UART_RX_STATUS`  | r     | r         | 1B     | u8       |           | Non-zero when there is data to read.                                    |
| `UART_RX_DATA`    | r     | r         | 1B     | u8       |           | Data to read. Only the LSB contains data.                               |
| `UART_RX_BYTES`   | r     | r         | 4B     | u32      |           | Number of bytes received from the host and not yet read by the SW or FW.|
| `UART_TX_STATUS`  | r     | r         | 1B     | u8       |           | Non-zero when it's OK to write data to send.                            |
| `UART_TX_DATA`    | w     | w         | 1B     | u8       |           | Data to send. Only the LSB contains data.                               |
| `TOUCH_STATUS`    | r/w   | r/w       |        |          |           | `TOUCH_STATUS_EVENT_BIT` is 1 when touched. After detecting a touch event (reading a 1), write anything here to acknowledge the event. |
| `FW_RAM`          | r/w   | invisible | 2 kiB  | u8[2048] |           | Firmware-only RAM.                                                      |
| `UDI`             | r     | invisible | 8B     | u64      |           | Unique Device ID (UDI).                                                 |
| `QEMU_DEBUG`      | w     | w         |        | u8       |           | Debug console (only in QEMU)                                            |
| `NAME0`           | r     | r         | 4B     | char[4]  | "tk1 "    | ID of core/stick, first part.                                           |
| `NAME1`           | r     | r         | 4B     | char[4]  | "mkdf"    | ID of core/stick, second part.                                          |
| `VERSION`         | r     | r         | 4B     | u32      | 1         | Version of core/stick.                                                  |
| `SWITCH_APP`      | r/w   | r         | 1B     | u8       |           | Write anything here to trigger the switch to application mode. Reading returns `0` if TKey is in firmware mode, `0xffffffff` if in app mode. |
| `LED`             | r/w   | r/w       | 1B     | u8       |           | Controls the RGB color of the status indicator LED on TKey. Bit 0 is Blue, bit 1 is Green, and bit 2 is Red LED. |
| `GPIO`            | r/w   | r/w       | 1B     | u8       |           | Bits 0 and 1 contain the input level of GPIO 1 and 2. Bits 3 and 4 store the output level of GPIO 3 and 4. |
| `APP_ADDR`        | r/w   | r         | 4B     | u32      |           | App load address, stored by firmware so app can find itself in memory.  |
| `APP_SIZE`        | r/w   | r         | 4B     | u32      |           | App size, stored by firmware so app can read its own size.              |
| `BLAKE2S`         | r/w   | r         | 4B     | u32      |           | Function pointer to a BLAKE2S function in the firmware.                 |
| `CDI_FIRST`       | r/w   | r         | 32B    | u8[32]   |           | The computed Compound Device Identifier (CDI).                          |
| `CDI_LAST`        |       | r         |        |          |           | Last word of CDI.                                                       |
| `RAM_ASLR`        | w     | invisible | 4B     | u32      |           | Seed value for the RAM randomization.                                   |
| `RAM_SCRAMBLE`    | w     | invisible | 4B     | u32      |           | Data scrambling seed value for the RAM.                                 |
| `CPU_MON_CTRL`    | w     | w         | 4B     | u32      |           | Bit 0 enables CPU execution monitor. Can't be unset. Lock addresses.    |
| `CPU_MON_FIRST`   | w     | w         | 4B     | u32      |           | First address of the area monitored for execution attempts.             |
| `CPU_MON_LAST`    | w     | w         | 4B     | u32      |           | Last address of the area monitored for execution attempts.              |

[^3]: Each word of the UDS can only be read *once* after TKey has started.
