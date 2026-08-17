<div align="center">

# IPv6 Addressing

*Chapter P — IPv6 Networking | Topic P.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

IPv6 replaces IPv4's 32-bit addresses with 128-bit ones and changes several habits learned in Chapter B: there is no broadcast, every interface has a link-local address, and ARP is replaced by Neighbor Discovery. This topic covers the address format and types.

## Why This Matters

More ISPs deliver IPv6 every year, and devices prefer it when it is available. Reading IPv6 addresses and knowing which ones are local, global, or for documentation is the starting point for P.02–P.04.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses and Subnet Masks](../../B-network-fundamentals/04-ipv4-addresses-and-subnet-masks/README.md) | Prefixes and masks. |
| [B.06 — Public and Private IP Addresses](../../B-network-fundamentals/06-public-and-private-ip-addresses/README.md) | Special ranges. |

## Theory

### 128 bits

IPv6 increases the address size from 32 to 128 bits. [1] Some IPv4 header fields were dropped or made optional. [1] The Hop Limit field replaces TTL and is decreased by one at each forwarding node. [1] Routers do not fragment IPv6 packets; only the source does. [1]

### Writing addresses

An IPv6 address is written as eight groups of up to four hexadecimal digits, `x:x:x:x:x:x:x:x`. [2] One run of zero groups can be replaced by `::`, once per address. [2]

```text
2001:DB8:0:0:8:800:200C:417A  =  2001:DB8::8:800:200C:417A
```

[2] Prefixes use the same `/length` notation as CIDR. [2]
