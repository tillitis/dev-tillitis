---
title: Tkey Unlocked
bookCollapseSection: true
weight: 6
---

# Tkey Unlocked

The unlocked version of the TKey comes with an FPGA without
configuration, i.e., it does not have a bitstream programmed. This
means that you can provision the TKey yourself and then either: 1) use
it as your everyday security key or 2) use it for hardware or firmware
development and re-provision as you go. These pages aims to provide
the tools and instructions needed to provision a TKey.

<img src="../images/tp1-tkey-min.png" alt="TKey Unlocked in tp1" title="TKey Unlocked in tp1" width="85%">

Why would one want TKey Unlocked?
1. You want to own and provision your security key yourself. You don't
have to put the same amount of trust in Tillitis.
2. You want to experiment, test, or develop on the FPGA or the
firmware and reprogram the FPGA multiple times.

If you want a Tkey that works right out of the box or just to develop
TKey device apps on, the [ordinary
Tkey](https://shop.tillitis.se/products/tkey) is probably right for
you.

{{< hint danger >}}
The below instructions require some computer knowledge and are only
recommended for advanced users. If you do not know what you are doing,
you can either brick your TKey or create a TKey that has security
vulnerabilities.

If you are unsure, please reach out to Tillitis and friends at the
`#tillitis` channel on the [OFTC IRC network](https://www.oftc.net/)
or `#tillitis:matrix.org` on [Matrix](https://matrix.org/).

{{< /hint >}}

In order to provision a TKey Unlocked, you need:
- [an FPGA bitstream binary](build/)
- [TP1 programming board](/TP1)
- [TKey
  Unlocked](https://shop.tillitis.se/products/tkey-not-provisioned),
  i.e., a TKey where the NVCM has not been written
- preferably an encrypted filesystem where you generate
  and temporarily store the bitstream and UDS files
- Glue for the casing, see [casing](unlocked/casing)

The bitstream consists of:
- FPGA configuration
- firmware running on the RISC-V core
- Unique Device Secret (UDS)
- Unique Device Identifier (UDI)

Programming the above bitstream onto a TKey is what we call
"provisioning".
It consists of three steps:
1. Build the bitstream
2. Program the TKey
3. Remove traces of your secrets after build

There are two options when programming the TKey: either write to the
onboard SPI flash or the one-time-writable on-chip NVCM.

The SPI flash is located next to the FPGA. It can be re-flashed any
number of times, or at least until the flash memory wears out. This is
recommended when you want to experiment with the TKey hardware
platform, i.e., using it for hardware or firmware development.

The NVCM is located inside the FPGA. It can be written only once.
By enabling the security bit, it will not be possible to read any data
from the NVCM. The TKey is then “locked” with whatever was programmed.
This is the recommended choice if you want to provision the TKey for
everyday use.

{{< hint info  >}}
By following these steps and choosing to program to NVCM, the [threat
model](https://tillitis.se/tkey/threat-model/) is valid for Tkey
Unlocked.
{{< /hint >}}

## Arrival of Tkey
When a Tkey Unlocked is sent from Tillitis, it will contain an [LED
blink design](https://github.com/tillitis/tillitis-key1/tree/main/hw/production_test/application_fpga_test_gateware)
in the SPI flash. It will blink sequentially in red, green, blue and 
white until you program either the SPI flash or the NVCM. This happens
to be the last step of our production test suite and is a method for
you to know that the device in your hand has passed those tests.

It is recommended to power up the TKey once it is recieved to control this
behavior. If your device does not blink, you should contact Tillitis. 

{{< hint info >}}
**NOTE:**
TKeys not provisioned by Tillitis cannot be verified with TKey Device
Verification. It is, however, possible to follow the same process to
be able to later verify that your self-provisioned TKey has not been
tampered with. See [TKey
Verification](https://github.com/tillitis/tkey-verification) for more
information.
{{< /hint >}}

## Start

Start by looking over and setting up permissions for [the TKey
Programmer](tp1), then continue to [Build an FPGA
bitstream](unlocked/build).

After programming either the NVCM or the SPI flash, you should
protect your TKey Unlocked by [assembling the plastic
case](unlocked/casing). When
all the steps are done, you should have a functioning TKey!
