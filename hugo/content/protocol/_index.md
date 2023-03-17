---
title: Protocols
weight: 4
---

# Protocols

## Framing Protocol

Tillitis provides a framing protocol that allows TKey programs and client
programs to communicate with the TKey firmware and each other. 
It is optional to use the framing protocol for TKey programs but 
mandatory for client programs.

The TKey program initiates the communication with the client program.
The communication over the framing protocol, between the programs, 
is command-response based. This means that the TKey program sends 
a command to the client program, and the client program responds 
to the given command.

Commands are processed by TKey in order. If the host sends a new 
command before receiving a response to the previous command, 
it is the responsibility of the TKey program to determine to which 
command a received response belongs to.

Commands and responses are sent as frames with a limited length. 
The communicating programs decide the meaning of the command and 
response frames, and if the commands and responses are valid and make sense.

### Command Frame Format

A command frame consists of a single header byte followed by one or more
data bytes. The number of data bytes in the frame is given by the header
byte. The header byte also specifies the sending or receiving program
for the command.

The bits in the command header byte should be interpreted as:
* Bit [7] (1 bit). Reserved - possible protocol version.

* Bits [6..5] (2 bits). Frame ID tag.

* Bits [4..3] (2 bits). Endpoint number. VB: Can you use a more descriptive word than endpoint like program?
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
**must** be preserved in the corresponding response to the command.

### Command Frame Examples

Note that most of the examples below do not take into account that the 
particular program or FW command request typically occupies the first byte 
in the data (following the command header byte), which makes 1 byte
less available for command content.

VB: Can you replace endpoints?

These examples clarify endpoints and framing protocol commands:

* 0x13: A command to the FW in the application_fpga with 512 bytes of
  data. The data could for example be parts of an flication binary to
  be loaded into the program memory.

* 0x1a: A command to the TKey program running in the application_fpga
  with 32 bytes of data. The data could be a 32 byte challenge to be
  signed using a private key from a TKey.

### Response Frame Format
A response consists of a single header byte followed by one or more bytes.

The bits in the response header byte should be interpreted as follows:
* Bit [7] (1 bit). Reserved - possible protocol version.

* Bits [6..5] (2 bits). Frame ID tag.

VB: Can you replace endpoint?

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

Note that the ID in a response **must** be the same as in the received command.

### Response Frame Examples

Note that most of the examples below do not take into account that the 
particular program or FW response code typically occupies the first byte 
in the data (following the response header byte), which makes 1 byte
less available for response content.

* 0x14: An unsuccessful command to the FW in the application_fpga which
  responds with a single byte of data.

* 0x1b: A successful command to the program running in the
  application_fpga. The response contains 512 bytes of data, for example
  an EdDSA Ed25519 signature.

## Firmware Protocol

### Firmware Protocol Definition

The firmware commands and responses are built on top of the framing
protocol above.

The commands look like this:

| *Name*           | *Size (bytes)* | *Comment*                                |
|------------------|----------------|------------------------------------------|
| Header           | 1              | Framing protocol header including length |
|                  |                | of the rest of the frame.                |
| Command/Response | 1              | Any of the below commands or responses.  |
| Data             | n              | Any additional data.                     |

The responses might include a one byte status field where 0 is
`STATUS_OK` and 1 is `STATUS_BAD`.

Note that the integer types are little-endian (LE).

#### `FW_CMD_NAME_VERSION` (0x01)

Get the name and version of the stick.

#### `FW_RSP_NAME_VERSION` (0x02)

| *Name*  | *Size (bytes)* | *Comment*            |
|---------|----------------|----------------------|
| name0   | 4              | ASCII                |
| name1   | 4              | ASCII                |
| version | 4              | Integer version (LE) |

VB: Should it be "NULL" instead of "zeroed"? I may be utterly wrong.

In a bad response, the fields are zeroed.

#### `FW_CMD_LOAD_APP` (0x03)

| *Name*       | *Size (bytes)* | *Comment*           |
|--------------|----------------|---------------------|
| size         | 4              | Integer (LE)        |
| uss-provided | 1              | 0 = false, 1 = true |
| uss          | 32             | Ignored if above 0  |

Start an application loading session by setting the size of the
expected device application and a User-Supplied Secret, if
`uss-provided` is 1. Otherwise the `USS` is ignored.

#### `FW_RSP_LOAD_APP` (0x04)

Response to `FW_CMD_LOAD_APP`

| *Name* | *Size (bytes)* | *Comment*                   |
|--------|----------------|-----------------------------|
| status | 1              | `STATUS_OK` or `STATUS_BAD` |

#### `FW_CMD_LOAD_APP_DATA` (0x05)

| *Name* | *Size (bytes)* | *Comment*           |
|--------|----------------|---------------------|
| data   | 511            | Raw binary program data |

Load 511 bytes of raw program binary data into the TKey RAM. Should be sent
consecutively over the complete raw binary. (512 == largest frame
length minus the command byte).

#### `FW_RSP_LOAD_APP_DATA` (0x06)

Response to all but the ultimate `FW_CMD_LOAD_APP_DATA` commands.

| *Name* | *Size (bytes)* | *Comment*                |
|--------|----------------|--------------------------|
| status | 1              | `STATUS_OK`/`STATUS_BAD` |

#### `FW_RSP_LOAD_APP_DATA_READY` (0x07)

VB: What does un-keyed mean?

The response to the last `FW_CMD_LOAD_APP_DATA` is an
`FW_RSP_LOAD_APP_DATA_READY` with the un-keyed hash digest for the
application that was loaded. It allows the host to verify that the
application was correctly loaded. This means that the calculated CDI
will be correct given that the UDS has not been modified.

| *Name* | *Size (bytes)* | *Comment*                |
|--------|----------------|--------------------------|
| status | 1              | `STATUS_OK`/`STATUS_BAD` |
| digest | 32             | BLAKE2s(app)             |

#### `FW_CMD_GET_UDI` (0x08)

Ask for the Unique Device Identifier (UDI) of the TKey.

#### `FW_RSP_GET_UDI` (0x09)

Response to `FW_CMD_GET_UDI`.

| *name* | *size (bytes)* | *comment*                                           |
|--------|----------------|-----------------------------------------------------|
| status | 1              | `STATUS_OK`/`STATUS_BAD`                            |
| udi    | 4              | Integer (LE) with Reserved (4 bit), Vendor (2 byte),|
|        |                | Product ID (6 bit), Product Revision (6 bit)        |
| udi    | 4              | Integer serial number (LE)                          |


#### Get the name and version of the device

```
host ->
  u8 CMD[1 + 1];

  CMD[0].len = 1    // command frame format
  CMD[1]     = 0x01 // FW_CMD_NAME_VERSION

host <-
  u8 RSP[1 + 32]

  RSP[0].len  = 32   // command frame format
  RSP[1]      = 0x02 // FW_RSP_NAME_VERSION

  RSP[2..6]   = NAME0
  RSP[6..10]  = NAME1
  RSP[10..14] = VERSION

  RSP[14..]   = 0
```

#### Load an application

```
host ->
  u8 CMD[1 + 512];

  CMD[0].len = 512  // command frame format
  CMD[1]     = 0x03 // FW_CMD_LOAD_APP

  CMD[2..6]  = APP_SIZE

  CMD[6]     = USS supplied? 0 = false, 1 = true
  CMD[7..39] = USS
  CMD[40..]  = 0

host <-
  u8 RSP[1 + 4];

  RSP[0].len = 4    // command frame format
  RSP[1]     = 0x04 // FW_RSP_LOAD_APP

  RSP[2]     = STATUS

  RSP[3..]   = 0

repeat ceil(APP_SIZE / 511) times:
host ->
  u8 CMD[1 + 512];

  CMD[0].len = 512  // command frame format
  CMD[1]     = 0x05 // FW_CMD_LOAD_APP_DATA

  CMD[2..]   = APP_DATA (511 bytes of program data, pad with zeros)

host <-
  u8 RSP[1 + 4]

  RSP[0].len = 4    // command frame format
  RSP[1]     = 0x06 // FW_RSP_LOAD_APP_DATA

  RSP[2]     = STATUS

  RSP[3..]   = 0
```

Except response from last chunk of program data which is:

```
host <-
  u8 RSP[1 + 512]

  RSP[0].len = 512  // command frame format
  RSP[1]     = 0x07 // FW_RSP_LOAD_APP_DATA_READY

  RSP[2]     = STATUS

  RSP[3..35]   = program digest
  RSP[36..]    = 0
```
