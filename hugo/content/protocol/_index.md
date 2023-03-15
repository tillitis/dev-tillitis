---
title: Framing protocol
weight: 4
---

# Framing Protocol

We provide a framing protocol to communicate with the TKey firmware
and TKey programs. Programs can choose to use this protocol.

The communication is driven by the host and the protocol is
command-response based. The host sends a command, and the TK1 must
always send a response to a given command. Commands are processed by
the TK1 in order. If the host sends a new command before receiving a
response to the previous command, it is the responsibility of the host
to determine to which command a received response belongs to.

Commands and responses are sent as frames with a constrained set of
possible lengths. It is the endpoints that are communicating that
decides what the data in the command and response frames mean, and if
the commands and responses are valid and make sense.

## Command frame format

A command frame consists of a single header byte followed by one or more
data bytes. The number of data bytes in the frame is given by the header
byte. The header byte also specifies the endpoint for the command.

The bits in the command header byte should be interpreted as:
* Bit [7] (1 bit). Reserved - possible protocol version.

* Bits [6..5] (2 bits). Frame ID tag.

* Bits [4..3] (2 bits). Endpoint number.
0. HW in interface_fpga Unused.
1. HW in application_fpga
2. FW in application_fpga
3. SW (program) in application_fpga

* Bit [2] (1 bit). Unused. MUST be zero.

* Bits [1..0] (2 bits). Command data length.
0. 1 byte
1. 4 bytes
2. 32 bytes
3. 512 bytes

Note that the number of bytes indicated by the command data length field
does **not** include the command header byte. This means that a complete
command frame, with a header indicating a data length of 512 bytes, is
512+1 bytes in length.

Note that the host sets the frame ID tag. The ID tag in a given command
MUST be preserved in the corresponding response to the command.

## Command frame examples

Note that these examples mostly don't take into account that the first
byte in the data (following the command header byte) typically is
occupied by the particular app or FW command requested, so there is 1
byte less available for the "payload" of the command.

Some examples to clarify endpoints and commands:

* 0x13: A command to the FW in the application_fpga with 512 bytes of
  data. The data could for example be parts of an application binary to
  be loaded into the program memory.

* 0x1a: A command to the TKey program running in the application_fpga
  with 32 bytes of data. The data could be a 32 byte challenge to be
  signed using a private key derived in the TK1.

## Response frame format
A response consists of a single header byte followed by one or more bytes.

The bits in the response header byte should be interpreted as:
* Bit [7] (1 bit). Reserved - possible protocol version.

* Bits [6..5] (2 bits). Frame ID tag.

* Bits [4..3] (2 bits). Endpoint number.
0. HW in interface_fpga
1. HW in application_fpga
2. FW in application_fpga
3. SW (application) in application_fpga

* Bit [2] (1 bit). Response status.
0. OK
1. Not OK (NOK)

* Bits [1..0] (2 bits). Response data length.
0. 1 byte
1. 4 bytes
2. 32 bytes
3. 512 bytes


Note that the number of bytes indicated by the response data length field
does **not** include the response header byte. This means that a complete
response frame, with a header indicating a data length of 512 bytes, is
512+1 bytes in length.

Note that the ID in a response MUST be the same ID as was present in the
header of the command being responded to.

## Response frame examples

Note that these examples mostly don't take into account that the first
byte in the data (following the response header byte) typically is
occupied by the particular app or FW response code, so there is 1 byte
less available for the "payload" of the response.

* 0x14: An unsuccessful command to the FW in the application_fpga which
  responds with a single byte of data.

* 0x1b: A successful command to the application running in the
  application_fpga. The response contains 512 bytes of data, for example
  an EdDSA Ed25519 signature.
