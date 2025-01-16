---
title: USB Controller Protocol
weight: 3
---

## USB Controller Protocol

To communicate between the USB controller and the UART on the FPGA we
use a small protocol to indicate the mode of the data transmission,
either CDC or HID.

| *Name*  | *Size*    | *Comment*                                    |
|---------|-----------|----------------------------------------------|
| Mode    | 1B        | Origin or destination USB endpoint (CDC/HID) |
| Length  | 1B        | Number of bytes following                    |
| Payload | See above | Actual data from or to app                   |

Origin and destination is either:

- CDC (0x40)
- HID (0x80)

*Note well*: This protocol is not visible on the client.
