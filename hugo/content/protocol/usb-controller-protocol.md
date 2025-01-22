---
title: USB Controller Protocol
weight: 3
---

## USB Controller Protocol

To communicate between the USB controller and the UART on the FPGA we
use a small protocol to indicate the USB endpoint on the client side.
There are three different endpoints:

- CTRL (0x20): A USB HID special debug pipe. Useful for debug prints.
- CDC (0x40): USB CDC-ACM, a serial port on the client.
- HID (0x80): A USB HID security token device, useful for FIDO-type
  applications.

A small protocol header should always come first in every frame:

| *Name*   | *Size*    | *Comment*                          |
|----------|-----------|------------------------------------|
| Endpoint | 1B        | Origin or destination USB endpoint |
| Length   | 1B        | Number of bytes following          |
| Payload  | See above | Actual data from or to app         |

*Note well*: This protocol is not visible on the client.
