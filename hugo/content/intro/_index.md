---
title: Introduction
weight: 1
---

# Introduction

The Tillitis TKey is a small computer in a USB stick form factor that
can run small uploaded programs. The purpose of the TKey is to provide
a secure environment for programs that provide some security function.
Some examples of such security functions are:

- TOTP token generators.
- Signing oracles.
- Secure random numbers.
- Encryption.

There is no way of storing programs. They have to be uploaded
every time you insert the TKey.

Specifications:

- 32 bit RISC-V CPU @ 18 MHz.
- Execution monitor.
- Hardware-assisted ASLR and RAM scrambling.
- 128 kiB RAM.
- 2 kiB firmware RAM.
- 6 kiB ROM.
- True random number generator.
- USB CDC over type C connector.
- Timer.
- Two level of hardware privilege modes: firmware mode/app mode.
- CPU-controlled LED
- No persistent storage.

The unique feature of the TKey is that it measures the TKey program
before starting it. The measurement (a hash digest using BLAKE2s),
combined with a Unique Device Secret (UDS) is used to derive a base
secret we call Compound Device Identifier (CDI) for the TKey program.

This means that if the TKey program is altered, the CDI will also
change. If the keys derived from the CDI are the same as the last time
the TKey program was loaded onto the same device, the program can be
trusted not to have been altered.

The UDS is unique per device. The same program uploaded to another
TKey device will result in a different CDI.

The key derivation can also be combined with a User Supplied Secret
(USS). This means that keys derived are both based on something the
user has: the specific device, and something the user knows: the USS.

```go
cdi = blake2s(UDS, blake2s(program), USS)
```

All of the TKey software, firmware, FPGA Verilog source code,
schematics and PCB design files are open source. Like all trustworthy
security software and hardware should be. This in itself makes it
different, as other security tokens utilize at least some closed
source hardware for security-critical operations.

## Getting started

* [tillitis-key1-apps repository](https://github.com/tillitis/tillitis-key1-apps),
  with client and TKey programs.
* [Toolchain setup](../tools/).
