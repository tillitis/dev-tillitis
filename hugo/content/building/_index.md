---
title: Building
weight: 2
---

# Building apps

## Toolchain

To build you need the `clang`, `llvm`, `lld`, `golang` packages
installed. Version 15 or later of LLVM/Clang is required (with riscv32
support and Zmmul extension). Ubuntu 22.10 (Kinetic) is known to have
this and work. Please see
[toolchain_setup.md](https://github.com/tillitis/tillitis-key1/blob/main/doc/toolchain_setup.md)
(in the tillitis-key1 repository) for detailed information on the
currently supported build and development environment.

## Building

Build everything:

```
$ make
```

If your available `objcopy` is anything other than the default
`llvm-objcopy`, then define `OBJCOPY` to whatever they're called on
your system.

The apps can be run both on the hardware TKey, and on a QEMU machine
that emulates the platform. In both cases, the host program (the
program that runs on your computer, for example `tkey-ssh-agent`) will
talk to the app over a serial port, virtual or real. There is a
separate section below which explains running in QEMU.

