---
title: Introduction
weight: 1
---

# Introduction

<img src="../images/tkey-case-interaction-points.png" alt="TKey with interaction points marked" title="TKey with interaction points marked" width="85%">

The Tillitis TKey is a small computer in a USB stick form factor that
can run small device applications. The purpose of the TKey is to be a
secure environment for applications that provide some kind of security
function. Some examples of such security functions are:

- Time-based one-time password (TOTP) token generators
- Digital signatures
- Secure random numbers
- Encryption

There is a flash chip with a very simple filesystem on the TKey from
the Castor release. It contains space for two pre-loaded apps and four
storage areas for device apps. On an end-user version the preloaded
apps are a loader app and a FIDO2 app.

All of the TKey software, firmware, FPGA Verilog source code,
schematics, and PCB design files are released under open
source/hardware licenses, like all trustworthy security software and
hardware should be. This, in itself, makes the TKey different, as
other security tokens use at least some closed source hardware for
security-critical operations.

## TKey specifications

- 32-bit RISC-V CPU running at 24 MHz
- Execution monitor
- Hardware-assisted address randomization and RAM scrambling
- 128 kiB RAM for TKey device applications
- 4 kiB firmware RAM
- 8 kiB ROM
- True random number generator
- USB CDC, FIDO, HID, and CCID endpoints over a Type-C connector
- Timer
- Two levels of hardware privilege modes: firmware mode and
  application mode
- CPU-controlled LED
- 1 MByte flash, 128 kByte accessible per application.

{{< hint info >}}
**Note well**: In the end-user version (not TKey Unlocked) the FPGA
configuration is locked down. This means you cannot change the FPGA
bitstream or read out the bitstream (or the Unique Device Secret, UDS)
from the configuration memory, even if you break the case and insert
it into a programmer board.
{{< /hint >}}

## Measured and verified boot and chaining apps

The TKey creates a secret for the device application before starting
it. It does this by computing a hash function over the entire app and
mixing it together with a Unique Device Secret (UDS) embedded in the
hardware and an optional User Supplied Secret, typically based on a
passphrase.

This means that the resulting secret is based on:

1. Something the user *has*, the specific TKey device.
2. Something the user *knows*, the User Supplied Secret.
3. The *integrity* of the software, the device app they are about to
   use.

If any of these three changes, the secret is also different. If the
secret and the resulting cryptographics keys are the same as before
it's guaranteed that this is the right device, with the right software
and that the user knows the passphrase.

This is the algorithm for the secret:

```go
secret = blake2s(UDS, domain + blake2s(device_app) + USS)
```

The UDS is used as a key in a keyed version of BLAKE2s. The `domain`
here is a domain separation between if the app was using measured boot
or was chained (explained below) and with or without USS.

We call this secret the Compound Device Identity (CDI). The TKey
unconditional measured boot is inspired by, but not exactly the same
as part of [TCG
DICE](https://trustedcomputinggroup.org/work-groups/dice-architectures/).

From this version the TKey also supports a form of verified boot in
order to be able to update device apps without losing key material.

In this mode the TKey uses measured boot as above to start a verifier
app, then verifies a signature over the next app against a public key,
and leaves an app digest and some data to the next app. Then it resets
the TKey.

After reset, the TKey will only accept and start and app which has the
already verified digest.

In this case, the CDI for the verified app is computed like this:

```go
CDI = blake2s(UDS, domain + blake2s(app[n-1]'s CDI, seed)¹ + USS)
```

¹ This part is actually computed by the firmware before actually doing
the reset in order not to leak the CDI over the reset. Neither
firmware/app on any side of the reset knows about the other's CDI.

UDS is used as the key in a keyed BLAKE2s. So is `app[n-1]'s CDI`.

The `seed` is, if using Tillitis'
[tkey-boot-verifier](https://github.com/tillitis/tkey-boot-verifier) a
digest of the vendor public key used.

The `domain`, again, is a domain separation between if the app was
using measured boot or was chained (explained below) and with or
without USS.

This means that the resuling secret/CDI is based on:

1. Something the user *has*, the specific TKey device.
2. Something the user *knows*, the User Supplied Secret.
3. The *integrity* of the verifier, the device app that verifies the
   app they are about to use.
4. The vendor public key.

Change any of those four and you create a different secret.

## Getting Started

* [Get started using your TKey](https://tillitis.se/getstarted).
* [Tools & Libraries](../tools/), setup and introduction for
  application developers.
* [Tkey Unlocked](../unlocked), instructions for the provisioning
process
