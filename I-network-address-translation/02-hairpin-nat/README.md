<div align="center">

# Hairpin NAT

*Chapter I — Network Address Translation (NAT) | Topic I.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

After I.01, the web server works from outside, but a LAN device that opens the router's public address fails. This topic explains why and fixes it with hairpin NAT, also called NAT loopback.

## Why This Matters

People use the same address or DDNS name (E.05) at home and away. Without hairpin NAT, it works only away from home, which looks like a random fault.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [I.01 — Port Forwarding with dst-nat](../01-port-forwarding-with-dst-nat/README.md) | The forward to fix. |

## Theory

### Why it fails

A LAN client connects to the router's WAN address. The dstnat rule rewrites the destination to the server, which is on the same LAN. The server sees the client's LAN address as the source and replies directly, bypassing the router. The client expected the reply from the WAN address, so it drops it.

### The fix

A `srcnat` masquerade rule for traffic from the LAN to the server makes the router the source, so the server's reply goes back through the router, which reverses both translations. [1] The manual's rule:

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade protocol=tcp src-address=10.0.0.0/24 dst-address=10.0.0.3 out-interface-list=LAN comment="Hairpin NAT"
```

[1]

The dstnat rule must also match connections coming from the LAN: rules limited with `in-interface-list=WAN` do not; rules using `dst-address-type=local` without an interface list, or a fixed `dst-address`, do. [1]

### Alternative: local DNS

A static DNS entry (E.03) that makes the name resolve to the server's LAN address inside the LAN avoids hairpin NAT entirely.

### How it connects

```text
I.01 forward ──► I.02 hairpin ──► E.03 split DNS as the alternative
```

## Lab Environment

| Item | Value |
| --- | --- |
| Starting state | I.01 rule in place, server `192.168.88.50:80` |
| Client | Second LAN device, e.g. a phone on Wi-Fi |
