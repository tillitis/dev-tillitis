---
title: Memory Map
weight: 3
---

# Memory Map

An overview of the memory of the TKey. See [Hardware](/hw) for how to
use the hardware functions.

## Addressing

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

| *name*             | *prefix* | *address length*                     |
|--------------------|----------|--------------------------------------|
| ROM                | 0b00     | 30 bit address                       |
| RAM                | 0b01     | 30 bit address                       |
| reserved           | 0b10     |                                      |
| Hardware functions | 0b11     | 6 bits for core select, 24 bits rest |

## Address Prefixes

The first 8 bits in a 32-bit address.

| *name*  | *prefix*                        |
|---------|---------------------------------|
| ROM     | 0x00                            |
| RAM     | 0x40                            |
| TRNG    | 0xc0                            |
| TIMER   | 0xc1                            |
| UDS     | 0xc2                            |
| UART    | 0xc3                            |
| TOUCH   | 0xc4                            |
| FW\_RAM | 0xd0                            |
| SYSCALL | 0xe1                            |
| QEMU    | 0xfe Not used in real hardware! |
| TK1     | 0xff                            |

## Hardware functions

The hardware functions begins at `0xc000_0000` but please use the
constants in
[tk1_mem.h](https://github.com/tillitis/tkey-libs/blob/main/include/tkey/tk1_mem.h)
in the `tkey-libs` repository if you can.

*Note*: The address in the table below is the word address. The data might
be available only in the first bits or maybe the LSB of the word. Also
note that some data is readable only by word, not byte addressing.
Note the description.

In the following table **r** mean read, **r/w** means read and write
access, **i** means invisible.

| *Name*                       | *Word Address* | *FW* | *Syscall* | *App* | *Description*                                                                                                                                      |
|------------------------------|----------------|------|-----------|-------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| `TK1_FW_RAM_BASE`            | 0xd0000000     | r/w  | r/w       | i     | Start of firmware RAM (4 kiB).                                                                                                                     |
| `TK1_MMIO_TRNG_STATUS`       | 0xc0000024     | r    | r         | r     | If bit 1 set `TRNG_ENTROPY` is ready to be read.                                                                                                   |
| `TK1_MMIO_TRNG_ENTROPY`      | 0xc0000080     | r    | r         | r     | A word of entropy.                                                                                                                                 |
| `TK1_MMIO_TIMER_CTRL`        | 0xc1000020     | r/w  | r/w       | r/w   | If bit 0 in `TK1_MMIO_TIMER_STATUS` is 0, setting bit 0 starts the timer. If bit 0 in `TK1_MMIO_TIMER_STATUS` is 1, setting bit 1 stops the timer. |
| `TK1_MMIO_TIMER_STATUS`      | 0xc1000024     | r    | r         | r     | Bit 0 is 1 when the timer is running.                                                                                                              |
| `TK1_MMIO_TIMER_PRESCALER`   | 0xc1000028     | r/w  | r/w       | r/w   | Prescaler init value. Write blocked when running.                                                                                                  |
| `TK1_MMIO_TIMER_TIMER`       | 0xc100002c     | r/w  | r/w       | r/w   | Timer init or current value while running. Write blocked when running.                                                                             |
| `TK1_MMIO_UDS_FIRST`         | 0xc2000040     | r    | i         | i     | First word of Unique Device Secret key. 8 words. Word access only. **Note:** Only readable once per power up.                                      |
| `TK1_MMIO_UDS_LAST`          | 0xc200005c     | r    | i         | i     | Last word of the UDS. **Note:** Only readable once per power up.                                                                                   |
| `TK1_MMIO_UART_RX_STATUS`    | 0xc3000080     | r    | r         | r     | Non-zero when there is data to read.                                                                                                               |
| `TK1_MMIO_UART_RX_DATA`      | 0xc3000084     | r    | r         | r     | Data to read. Only the LSB of the word contains data.                                                                                              |
| `TK1_MMIO_UART_RX_BYTES`     | 0xc3000088     | r    | r         | r     | Number of unread bytes received from client.                                                                                                       |
| `TK1_MMIO_UART_TX_STATUS`    | 0xc3000100     | r    | r         | r     | Non-zero when it's OK to write data to send.                                                                                                       |
| `TK1_MMIO_UART_TX_DATA`      | 0xc3000104     | w    | w         | w     | Data to send. Only the LSB of the word will be sent.                                                                                               |
| `TK1_MMIO_TOUCH_STATUS`      | 0xc4000024     | r    | r         | r     | Bit 0 is 1 when touched. After detecting a touch event (reading a 1), write anything here to acknowledge the event.                                |
| `TK1_MMIO_SYSCALL`           | 0xe1000000     | w    | w         | w     | Write `1` to trigger a syscall.                                                                                                                    |
| `TK1_MMIO_QEMU_DEBUG`        | 0xfe001000     | w    | w         | w     | Debug console (only in QEMU). Only the LSB of the word will be sent.                                                                               |
| `TK1_MMIO_TK1_NAME0`         | 0xff000000     | r    | r         | r     | TKey FPGA design name, first part, default "tk1".                                                                                                  |
| `TK1_MMIO_TK1_NAME1`         | 0xff000004     | r    | r         | r     | TKey FPGA design name, second part, default "mkdf".                                                                                                |
| `TK1_MMIO_TK1_VERSION`       | 0xff000008     | r    | r         | r     | Version of FPGA design (See UDI for product version or serial)                                                                                     |
| `TK1_MMIO_TK1_LED`           | 0xff000024     | r/w  | r/w       | r/w   | Controls the RGB color of the status indicator LED on TKey. Bit 0 is Blue, bit 1 is Green, and bit 2 is Red LED.                                   |
| `TK1_MMIO_TK1_GPIO`          | 0xff000028     | r/w  | r/w       | r/w   | Bits 0 and 1 contain the input level of GPIO 1 and 2. Bits 2 and 3 store the output level of GPIO 3 and 4.                                         |
| `TK1_MMIO_TK1_APP_ADDR`      | 0xff000030     | r/w  | r/w       | r     | App load address, stored by firmware so app can find itself in memory.                                                                             |
| `TK1_MMIO_TK1_APP_SIZE`      | 0xff000034     | r/w  | r/w       | r     | App size, stored by firmware so app can read its own size.                                                                                         |
| `TK1_MMIO_TK1_CDI_FIRST`     | 0xff000080     | r/w  | r/w       | r     | The computed Compound Device Identifier (CDI). 8 words.                                                                                            |
| `TK1_MMIO_TK1_CDI_LAST`      | 0xff00009c     | r/w  | r/w       | r     | Last word of CDI.                                                                                                                                  |
| `TK1_MMIO_TK1_UDI_FIRST`     | 0xff0000c0     | r    | r         | i     | First word of Unique Device ID (UDI). 2 words. Set during provisioning.                                                                            |
| `TK1_MMIO_TK1_UDI_LAST`      | 0xff0000c4     | r    | r         | i     | Last word of UDI                                                                                                                                   |
| `TK1_MMIO_TK1_RAM_ADDR_RAND` | 0xff000100     | w    | w         | i     | Seed word for the RAM address randomization.                                                                                                       |
| `TK1_MMIO_TK1_RAM_DATA_RAND` | 0xff000104     | w    | w         | i     | Seed word for the RAM data randomization.                                                                                                          |
| `TK1_MMIO_TK1_CPU_MON_CTRL`  | 0xff000180     | w    | w         | w     | Bit 0 enables Security Monitor. Can't be unset. Locks the area between the addresses set in `TK1_CPU_MON_FIRST` and `TK1_CPU_MON_LAST`.            |
| `TK1_MMIO_TK1_CPU_MON_FIRST` | 0xff000184     | w    | w         | w     | Start address (32-bit) of the RAM area monitored for execution attempts.                                                                           |
| `TK1_MMIO_TK1_CPU_MON_LAST`  | 0xff000188     | w    | w         | w     | Last address (32-bit) of the RAM area monitored for execution attempts.                                                                            |
| `TK1_MMIO_TK1_SYSTEM_RESET`  | 0xff0001C0     | w    | w         | i     | Write `1` to reset the FPGA.                                                                                                                       |
| `TK1_MMIO_TK1_SPI_EN`        | 0xff000200     | w    | w         | i     | Write `1` to enable the SPI-master, `0` to disable. The chip select pin will go low when the SPI-master is enabled and high when disabled.         |
| `TK1_MMIO_TK1_SPI_XFER`      | 0xff000204     | r/w  | r/w       | i     | Write to start an SPI byte transfer. Read to get the SPI transfer status. If the value read is not `0` a new transfer can be started.              |
| `TK1_MMIO_TK1_SPI_DATA`      | 0xff000208     | r/w  | r/w       | i     | Write to load byte to send. Read to get received byte.                                                                                             |
