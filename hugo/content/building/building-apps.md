+++
title = "Building apps"
date = 2023-01-25T16:19:52+01:00
weight = 2
+++

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

