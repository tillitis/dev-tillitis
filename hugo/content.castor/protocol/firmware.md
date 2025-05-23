---
title: Firmware Protocol
weight: 2
---

## Firmware Protocol

### Firmware Protocol Definition

The firmware commands and responses are built on top of the [framing
protocol](/protocol/framing_protocol/).

The commands look like this:

| *Name*           | *Size (bytes)* | *Comment*                                                          |
|------------------|----------------|--------------------------------------------------------------------|
| Header           | 1              | Framing protocol header including length of the rest of the frame. |
| Command/Response | 1              | Any of the below commands or responses.                            |
| Data             | n              | Any additional data.                                               |

The responses might include a one byte status field where 0 is
`STATUS_OK` and 1 is `STATUS_BAD`.

Note that the integer types are little-endian (LE).

#### `FW_CMD_NAME_VERSION` (0x01)

Get the name and version of the stick.

#### `FW_RSP_NAME_VERSION` (0x02)

| *Name*  | *Size (bytes)* | *Comment*            |
|---------|----------------|----------------------|
| Name0   | 4              | ASCII                |
| name1   | 4              | ASCII                |
| version | 4              | Integer version (LE) |

In a bad response the fields will be all zeros (0x00 bytes).

#### `FW_CMD_LOAD_APP` (0x03)

| *Name*       | *Size (bytes)* | *Comment*           |
|--------------|----------------|---------------------|
| size         | 4              | Integer (LE)        |
| uss-provided | 1              | 0 = false, 1 = true |
| uss          | 32             | Ignored if above 0  |

Start an application loading session by setting the size of the
expected device application and a User-Supplied Secret, if
`uss-provided` is 1. Otherwise the `uss` is ignored.

#### `FW_RSP_LOAD_APP` (0x04)

Response to `FW_CMD_LOAD_APP`

| *Name* | *Size (bytes)* | *Comment*                   |
|--------|----------------|-----------------------------|
| status | 1              | `STATUS_OK` or `STATUS_BAD` |

#### `FW_CMD_LOAD_APP_DATA` (0x05)

| *name* | *size (bytes)* | *comment*           |
|--------|----------------|---------------------|
| data   | 127            | Raw binary app data |

Load 127 bytes of raw app binary into the TKey RAM. Should be sent
consecutively over the complete raw binary. (128 == largest frame
length minus the command byte).

#### `FW_RSP_LOAD_APP_DATA` (0x06)

Response to all but the ultimate `FW_CMD_LOAD_APP_DATA` commands.

| *Name* | *Size (bytes)* | *Comment*                |
|--------|----------------|--------------------------|
| status | 1              | `STATUS_OK`/`STATUS_BAD` |

#### `FW_RSP_LOAD_APP_DATA_READY` (0x07)

The response to the last `FW_CMD_LOAD_APP_DATA` is an
`FW_RSP_LOAD_APP_DATA_READY` with the BLAKE2s-256 hash digest (no
secret key) for the application that was loaded. It allows the client
application on the host to verify that the application was correctly
loaded. This means that the calculated CDI will be correct, given that
the UDS has not been modified.

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

It's recommended to check if the TKey is in firmware mode before
attempting to load a device application. Typically this firmware probe
is implemented by sending `FW_CMD_NAME_VERSION` with a header byte
indicating endpoint 2, firmware.

We encourage device app developers to support a firmware probe and
reply `NOK` to anything that comes with a header byte for endpoint 2.

The sequence looks like this:

{{< mermaid class="optional" >}}
sequenceDiagram
	autonumber

	participant c as Client
	participant t as TKey

	c->>t: Running firmware?
	t->>c: Yes!
	c->>t: FW_CMD_LOAD_APP(size, USS)
    t->>c: FW_RSP_LOAD_APP

        loop Until entire device app sent
        	c->>t: FW_CMD_LOAD_APP_DATA(app block)
            t->>c: FW_RSP_LOAD_APP_DATA || FW_RSP_LOAD_APP_DATA_READY
        end
{{< /mermaid >}}

In detail, after the firmware probe:

```
host ->
  u8 CMD[1 + 128];

  CMD[0].len = 128  // command frame format
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

repeat ceil(APP_SIZE / 127) times:
host ->
  u8 CMD[1 + 128];

  CMD[0].len = 128  // command frame format
  CMD[1]     = 0x05 // FW_CMD_LOAD_APP_DATA

  CMD[2..]   = APP_DATA (127 bytes of app data, pad with zeros)

host <-
  u8 RSP[1 + 4]

  RSP[0].len = 4    // command frame format
  RSP[1]     = 0x06 // FW_RSP_LOAD_APP_DATA

  RSP[2]     = STATUS

  RSP[3..]   = 0
```

Except response from last chunk of app data which is:

```
host <-
  u8 RSP[1 + 128]

  RSP[0].len = 128  // command frame format
  RSP[1]     = 0x07 // FW_RSP_LOAD_APP_DATA_READY

  RSP[2]     = STATUS

  RSP[3..35]   = app digest
  RSP[36..]    = 0
```
