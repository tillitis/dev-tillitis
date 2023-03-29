---
title: Introduction
weight: 1
---

# Introduction

The Tillitis TKey is a small computer in a USB stick form factor that
can run small applications which are loaded onto it (called device
applications). The purpose of TKey is to be a secure environment for
applications that provide security function. Some examples of such
security functions are:

- Time-based one-time password (TOTP) token generators
- Signing oracles
- Secure random numbers
- Encryption

There is no way of storing a device application (or any other data) on
the TKey. A device app has to be loaded onto the TKey every time you
plug it in.

TKey specifications:

- 32-bit RISC-V CPU running at 18 MHz
- Execution monitor
- Hardware-assisted ASLR and RAM scrambling
- 128 kiB RAM for TKey device applications
- 2 kiB firmware RAM
- 6 kiB ROM
- True random number generator
- USB CDC (Communications Device Class) over a Type-C connector
- Timer
- Two levels of hardware privilege modes: firmware mode and application mode
- CPU-controlled LED
- No persistent storage

A unique feature of the TKey is that it measures the loaded device
application before starting it. A hash digest measurement (using
BLAKE2s) combined with a Unique Device Secret (UDS) make up a base
secret we call Compound Device Identifier (CDI) for use by the TKey
device app.

If the TKey device app is altered, the CDI is also changed. If the
keys derived from the CDI are the same as the last time the given
device app was loaded onto the same TKey, the device app can be
trusted not to have been altered.

The UDS is unique per TKey. The same device app loaded onto another
TKey results in a different CDI.

The key derivation can also include a User Supplied Secret (USS). Then
the keys are based on both something the user has -- the specific TKey
-- and something the user knows -- the USS.

This is the algorithm for the CDI:
```go
cdi = blake2s(UDS, blake2s(device_app), USS)
```

All of the TKey software, firmware, FPGA Verilog source code,
schematics and PCB design files are released under open
source/hardware licenses like all trustworthy security software and
hardware should be. This in itself makes it different, as other
security tokens use at least some closed source hardware for
security-critical operations.

## Getting Started

* [tillitis-key1-apps repository](https://github.com/tillitis/tillitis-key1-apps),
  with TKey client and device applications.
* [Tools & Libraries](../tools/), setup and introduction for
  application developers.
