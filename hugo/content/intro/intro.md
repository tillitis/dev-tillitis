+++
title = "Introduction"
date = 2023-01-25T16:19:52+01:00
weight = 2
+++

The Tillitis TKey is a small computer in a USB stick form factor with
a single USB C connector.

The CPU is a 32-bit RISC-V (RV32IC_Zmmul, basically RV32IMC without
division). Main RAM is 128 kiB.

The TKey has two modes of operation: *firmware mode* and *application
mode*. The firmware mode has the responsibility of receiving,
measuring, loading, and starting the application. Application mode is
a more locked down mode. There is no way of getting back to firmware
mode after switching to application mode.

