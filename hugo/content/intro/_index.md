---
title: Introduction
weight: 1
---

# Introduction

Tillitis' TKey is a small computer in a USB stick form factor that
can run small programs which are uploaded to it. The purpose of 
TKey is to be a secure environment for programs that provide 
security function. Some examples of such security functions are:

- Time-based one-time password (TOTP) token generators
- Signing oracles
- Secure random numbers
- Encryption

There is no way of storing programs on TKey. Programs have to be uploaded
to TKey every time you insert the TKey.

TKey specifications:

- 32 bit RISC-V CPU @ 18 MHz
- Execution monitor
- Hardware-assisted ASLR and RAM scrambling
- 128 kB RAM for TKey programs
- 2 kB firmware RAM
- 6 kB ROM
- True random number generator
- USB communications device class (CDC) over type C connector
- Timer
- Two level of hardware privilege modes: firmware mode/program mode
- CPU-controlled LED
- No persistent storage

A unique feature of the TKey is that it measures each TKey program
before starting it. A hash digest measurement (using BLAKE2s)
combined with a Unique Device Secret (UDS) make up a base
secret we call Compound Device Identifier (CDI) for the TKey program.

If the TKey program is altered, the CDI is also changed. If the keys 
derived from the CDI are the same as the last time the given 
TKey program was loaded onto the same TKey, the program can be
trusted not to have been altered.

The UDS is unique per TKey. The same program uploaded to another
TKey results in a different CDI.

The derived keys can also be combined with a User Supplied Secret
(USS). Then the keys are based on both something the
user has - the specific TKey - and something the user knows - the USS.

This is the algorithm for the CDI:
```go
cdi = blake2s(UDS, blake2s(program), USS)
```

All of the TKey software, firmware, FPGA Verilog source code,
schematics, and PCB design files are open source. Like all trustworthy
security software and hardware should be. This in itself makes it
different, as other security tokens utilize at least some closed
source hardware for security-critical operations.

## Getting Started

* [tillitis-key1-apps repository](https://github.com/tillitis/tillitis-key1-apps),
  with TKey programs and client programs.
  
  VB: Write an explaination like for previous bullet.
* [Toolchain setup](../tools/).
