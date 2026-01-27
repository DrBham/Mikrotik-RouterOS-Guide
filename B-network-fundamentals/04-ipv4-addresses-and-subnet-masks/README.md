<div align="center">

# IPv4 Addresses and Subnet Masks

*Chapter B — Network Fundamentals & IP Addressing | Topic B.04*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

An IPv4 address tells two things at once: which network a device is on, and which device it is on that network. The subnet mask is what separates the two. This topic explains the address, the mask, the prefix notation (`/24`), and the addresses that may not be given to a host.

## Why This Matters

`192.168.88.1/24` appears in almost every topic so far. Reading it correctly answers practical questions: which addresses can I give my devices, can this laptop reach that one directly, and why does a wrong mask break a network that "looks" correct.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.03 — Ethernet, MAC Addresses, and ARP](../03-ethernet-mac-and-arp/README.md) | Direct delivery inside one network. |

## Theory

### A 32-bit address

An IPv4 address is a fixed-length field of four octets, 32 bits. [1] It is written as four decimal numbers, one per octet, separated by dots. `192.168.88.1` in binary is:

```text
192      .168      .88       .1
11000000 .10101000 .01011000 .00000001
```

### The address mask

RFC 1122 describes the address mask as a 32-bit mask that selects the network part of an address (including any subnet part). [2] The remaining bits identify the host. RFC 950 recommends that the network bits be contiguous and start from the most significant end. [3] A mask of `255.255.255.0` is 24 one-bits followed by 8 zero-bits:

```text
Address  11000000.10101000.01011000.00000001   192.168.88.1
Mask     11111111.11111111.11111111.00000000   255.255.255.0
         └──────── network (24 bits) ────────┘ └ host ┘
```

### Prefix notation

CIDR writes the mask as a prefix length: the address, a slash, and the number of network bits from 0 to 32. [4] `192.168.88.1/24` means the same as address `192.168.88.1` with mask `255.255.255.0`. A `/24` contains 256 addresses. [4]

| Prefix | Mask | Addresses |
| --- | --- | --- |
| /24 | 255.255.255.0 | 256 |
| /25 | 255.255.255.128 | 128 |
| /30 | 255.255.255.252 | 4 |
| /32 | 255.255.255.255 | 1 |

Each extra prefix bit halves the size of the block, which is just binary arithmetic.

### Same network or not

A host decides whether a destination is local by applying its mask to both addresses. If the network bits match, the destination is on the connected network and the packet is sent directly. If not, the destination can only be reached through a gateway. [2]

| Laptop | Destination | Same network? |
| --- | --- | --- |
| 192.168.88.254/24 | 192.168.88.1 | Yes: send directly (ARP for its MAC) |
| 192.168.88.254/24 | 8.8.8.8 | No: send to the gateway (B.07) |

### Addresses a host may not use

A host-number field may not be all zeros or all ones, except in the special cases RFC 1122 lists. [2] Two of those cases matter every day:

| Form | Meaning [2] | In 192.168.88.0/24 |
| --- | --- | --- |
| Host part all ones | Directed broadcast to that network; never a source address | `192.168.88.255` |
| All bits ones | Limited broadcast: reaches every host on the physical network, never forwarded | `255.255.255.255` |
| `127.x.x.x` | Loopback inside a host; must never appear outside it | — |

With the network address (host part all zeros) and the broadcast address taken, a `/24` leaves 254 addresses for devices: `.1` to `.254`.

### How it connects

```text
B.04 address + mask ──► B.05 split networks into subnets ──► B.06 which ranges are private ──► B.07 send to a gateway when "not the same network"
```
