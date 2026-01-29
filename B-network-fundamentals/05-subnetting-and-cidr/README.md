<div align="center">

# Subnetting and CIDR

*Chapter B — Network Fundamentals & IP Addressing | Topic B.05*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Subnetting splits one block of addresses into smaller networks; CIDR describes any block by its prefix length and lets many blocks be summarised as one. This topic shows how to size a subnet, split a `/24`, and combine blocks into a larger prefix.

## Why This Matters

Later chapters give each part of a network its own subnet: a LAN, a guest Wi-Fi, a point-to-point link between two routers, a VLAN per department. Choosing the right prefix length avoids both running out of addresses and wasting them, and the same arithmetic explains how routers pick the most specific route.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses and Subnet Masks](../04-ipv4-addresses-and-subnet-masks/README.md) | Masks and prefix notation. |

## Theory

### From classes to prefixes

Early IPv4 had only three network sizes: Class A, B, and C. CIDR deprecated that system in favour of classless prefixes, so the network size is no longer implied by the first bits of the address. [1] With explicit prefix lengths, a block can be any power-of-two size from 1 to 2^32 addresses. [1] Because the mask can no longer be guessed from the address, routing protocols must carry it explicitly. [1]

### Block sizes

RFC 4632 lists every prefix size. [1] The small ones are the ones you will use most:

| Prefix | Addresses | Note [1] |
| --- | --- | --- |
| /32 | 1 | "host route" |
| /31 | 2 | "p2p link" |
| /30 | 4 | |
| /29 | 8 | |
| /28 | 16 | |
| /27 | 32 | |
| /26 | 64 | |
| /25 | 128 | |
| /24 | 256 | legacy Class C size |

### Subnetting: borrowing host bits

RFC 950 defines subnetting as taking bits from the host part of an address to number subnets, identified by the mask. [2] Each borrowed bit doubles the number of subnets and halves their size.

Splitting `192.168.88.0/24` into four `/26` subnets (2 borrowed bits):

| Subnet | Network | First usable | Last usable | Broadcast |
| --- | --- | --- | --- | --- |
| 1 | 192.168.88.0/26 | .1 | .62 | .63 |
| 2 | 192.168.88.64/26 | .65 | .126 | .127 |
| 3 | 192.168.88.128/26 | .129 | .190 | .191 |
| 4 | 192.168.88.192/26 | .193 | .254 | .255 |

Each subnet loses its network address and its broadcast address, as in B.04. [3]

### How to size a subnet

Pick the smallest block whose usable addresses cover the devices plus the router:

| Need | Smallest block | Usable |
| --- | --- | --- |
| 2 routers on a link | /30 | 2 |
| 10 devices | /28 | 14 |
| 50 devices | /26 | 62 |
| 200 devices | /24 | 254 |
