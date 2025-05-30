---
title: Protocols
bookCollapseSection: true
weight: 4
---

# Protocols

The TKey projects uses some different protocols. First, there is
obviously the protocol the firmware uses to load and start a device
app. It is, in turn, based on a simple framing header and convention
that you are free to use in your own apps as well, if you like.

Since the TKey supports both several classes with an endpoint each, we
also use another small protocol, the USB Mode Protocol, to indicate
the endpoint mode of each packet.

