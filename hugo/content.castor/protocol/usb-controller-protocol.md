---
title: USB Mode Protocol
weight: 3
---

## USB Mode Protocol

To communicate between the USB controller and the UART on the FPGA we
use a small protocol to indicate the USB endpoint on the client side.
There are three different endpoints:

- QEMU (0x02).
- CH552 (0x04).
- CDC (0x08): USB CDC-ACM, a serial port on the client.
- FIDO (0x10): A USB HID security token device, useful for FIDO-type
  applications.
- CCID (0x20): CCID "smart card".
- DEBUG (0x40): A USB HID special debug pipe. Useful for debug prints.

A small protocol header should always come first in every frame:

| *Name*   | *Size*    | *Comment*                          |
|----------|-----------|------------------------------------|
| Endpoint | 1B        | Origin or destination USB endpoint |
| Length   | 1B        | Number of bytes following          |
| Payload  | See above | Actual data from or to app         |

*Note well*: This protocol is not visible on the client.
