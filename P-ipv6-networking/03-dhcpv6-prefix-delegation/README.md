<div align="center">

# DHCPv6 Prefix Delegation

*Chapter P — IPv6 Networking | Topic P.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

ISPs that offer IPv6 usually delegate a whole prefix to the customer router with DHCPv6. The router keeps it in a pool and carves `/64` subnets for its LANs, which clients then configure with SLAAC. This topic builds that chain, with a CHR playing the ISP.

## Why This Matters

This is how real home and office IPv6 works. Without prefix delegation, the router has nothing to advertise to its LAN.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [P.02 — Neighbor Discovery and SLAAC](../02-neighbor-discovery-and-slaac/README.md) | Advertising a `/64`. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | DHCP in IPv4. |

## Theory

### Prefix delegation

DHCPv6 provides a mechanism to delegate IPv6 prefixes automatically. [1] With `request=prefix`, the RouterOS DHCPv6 client puts the delegated prefix into a pool named by `pool-name`. [2][3] `pool-prefix-length` sets the size of the pieces taken from it; it must be equal to or longer than the received prefix. [3]

### Using the pool

An address with `from-pool` takes a subnet from the pool: `::1/64` becomes `<prefix>::1/64`. [4] With `advertise=yes`, LAN clients use it for SLAAC (P.02). [4]

### Default route

DHCPv6 itself does not provide a gateway; `add-default-route=yes` on the DHCPv6 client adds `::/0` towards the DHCPv6 server, and it is off by default. [3] Often the ISP's router advertisement provides the default route instead.

### The manual's example

ISP side (R1): [2]

```routeros
/ipv6/pool add name=myPool prefix=2001:db8::/62 prefix-length=64
/ipv6/dhcp-server add prefix-pool=myPool disabled=no interface=to-CE-routers lease-time=3m name=server1
```

Customer side (CE): [2]

```routeros
/ipv6/dhcp-client add interface=to-R1 request=prefix pool-name=my-ipv6
/ipv6/address add address=::1/64 from-pool=my-ipv6 interface=to-clients advertise=yes
```

## Lab Environment

| Item | Value |
| --- | --- |
| ISP | CHR R1, interface `to-CE-routers`, `TODO` version |
| Customer | hAP lite or CHR, WAN `to-R1` (ether1), LAN `to-clients` (bridge) |
| Prefix | `2001:db8::/62` (documentation) |

Rename interfaces to match the manual, or adapt the names.
