---
title: Framing Protocol
weight: 1
---

## Framing Protocol

Tillitis provides a framing protocol for client applications to
communicate with the TKey firmware and device applications. Developers
can optionally use this framing protocol as a basis for their own TKey
device applications - but client applications *must* use it when
communicating with the firmware using the firmware protocol.

The communication is driven by the client application running on the
host computer and the protocol is command-response based. The client
app sends a command, and the TKey sends a response to the given
command.

Commands are processed by the TKey in order. If the client app sends a
new command before receiving a response to the previous command, it is
the responsibility of the client app to determine which command a
received response belongs to.

Commands and responses are sent as frames with a limited length. The
communicating parties decide the meaning of the command and response
frames, and whether the commands and responses are valid or not.

### Command Frame Format

A command frame consists of a single header byte followed by one or
more data bytes. The number of data bytes in the frame is given by the
header byte. The header byte also specifies the domain for the
command/response -- essentially indicating if it is sent to/by the
firmware, or a device application. Note that "domain" was earlier
referred to as an endpoint, but it has been renamed to distinguish it
from USB endpoints.





The bits in the command header byte should be interpreted as:

* Bit [7] (1 bit). Reserved - possible protocol version.

* Bits [6..5] (2 bits). Frame ID tag.

* Bits [4..3] (2 bits). Domain number.

  0. Reserved
  1. HW in application_fpga (unused)
  2. Firmware
  3. Device application

* Bit [2] (1 bit). Unused. MUST be zero.

* Bits [1..0] (2 bits). Command data length.

  0. 1 byte
  1. 4 bytes
  2. 32 bytes
  3. 128 bytes

Note that the number of bytes indicated by the command data length field
does **not** include the command header byte. This means that a complete
command frame, with a header indicating a data length of 128 bytes, is
128+1 bytes in length.

Note that the client application sets the frame ID tag. The ID tag in
a given command **must** be preserved in the corresponding response to
the command.

### Command Frame Examples

Note that most of the examples below do not take into account that the
particular app or firmware command request typically occupies the
first byte in the data (following the frame header byte), which makes
1 byte less available for actual command content.

These examples clarify domains and commands using the framing
protocol:

* 0x13: A command to the firmware with 128 bytes of data. The data
  could, for example, be parts of an application binary to be loaded
  into memory.

* 0x1a: A command to the application running with 32 bytes of data.
  The data could be a 32 byte challenge to be signed using the private
  key derived in the TKey.

### Response Frame Format

A response consists of a single header byte followed by one or more bytes.

The bits in the response header byte should be interpreted as follows:

* Bit [7] (1 bit). Reserved - possible protocol version.

* Bits [6..5] (2 bits). Frame ID tag.

* Bits [4..3] (2 bits). Domain number.

  0. Reserved
  1. HW in application_fpga (unused)
  2. Firmware
  3. Device application

* Bit [2] (1 bit). Response status.

  0. OK
  1. Not OK (NOK)

* Bits [1..0] (2 bits). Response data length.

  0. 1 byte
  1. 4 bytes
  2. 32 bytes
  3. 128 bytes


Note that the number of bytes indicated by the response data length field
does **not** include the response header byte. This means that a complete
response frame, with a header indicating a data length of 128 bytes, is
128+1 bytes in length.

Note that the ID in a response **must** be the same ID as was present
in the header of the command being responded to.

### Response Frame Examples

Note that most of the examples below do not take into account that the
particular app or firmware response request typically occupies the
first byte in the data (following the frame header byte), which makes
1 byte less available for actual response content.

* 0x14: An unsuccessful command to the firmware which responds with a
  single byte of data.

* 0x1b: A successful command to the device application running. The
  response contains 128 bytes of data, for example an EdDSA Ed25519
  signature.

