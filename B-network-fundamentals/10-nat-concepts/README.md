<div align="center">

# NAT Concepts

*Chapter B — Network Fundamentals & IP Addressing | Topic B.10*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Network Address Translation (NAT) rewrites the addresses in a packet as it passes through a router. It is how a whole LAN of private addresses (B.06) reaches the internet through one address, and how a service inside the LAN can be published to the outside. This topic explains basic NAT, NAPT, source NAT, destination NAT, and masquerade.

## Why This Matters

A.03 Step 7 was the moment the laptop could finally browse: wlan1 joined the NAT rule. Without NAT, the home router would receive packets from `192.168.88.x`, a network it knows nothing about, and the replies would never come back.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.06 — Public and Private IP Addresses](../06-public-and-private-ip-addresses/README.md) | Why private addresses cannot go to the internet. |
| [B.02 — OSI and TCP/IP Models](../02-osi-and-tcp-ip-models/README.md) | Transport ports. |

## Theory

### Why NAT exists

NAT is needed when a network's internal addresses cannot be used outside it, either because they are not valid there or because they must stay private. [1] RFC 2663 describes it as mapping addresses from one address realm to another, transparently to the end hosts. [1]

### Basic NAT and NAPT

| Type | How it works [2] |
| --- | --- |
| Basic NAT | Private addresses are mapped to a set of global addresses. Only as many inside hosts as there are global addresses can be outside at the same time. |
| NAPT (network address and port translation) | Each (private address, port) pair is mapped to (registered address, assigned port). Many hosts share one registered address. |

RFC 3022 describes the home and small-office case directly: the router has one address from the provider, and NAPT lets every node on the private network use it at the same time. [2] The same model allows inbound access by statically mapping a port of the registered address to an inside host. [2]

A connection is identified by source address, source port, destination address, and destination port. [1] The ports are what let NAPT tell the replies for different inside hosts apart.

```text
192.168.88.10:51000 ──►┐                    ┌── 203.0.113.5:40001 ──► server
                       ├─ NAPT on router ───┤
192.168.88.11:51000 ──►┘                    └── 203.0.113.5:40002 ──► server
```

### Source NAT and destination NAT

RouterOS names the two directions by chain [3]:

| Chain | Rewrites | Typical use |
| --- | --- | --- |
| `srcnat` | The source address of packets leaving the local network | LAN to internet |
| `dstnat` | The destination address of packets arriving for the local network | Port forwarding to an inside server |

### Masquerade and src-nat

| Action | When to use [3] |
| --- | --- |
| `masquerade` | The WAN address changes (DHCP, PPPoE). Connections are removed when the interface goes down. |
| `src-nat` with `to-addresses` | The WAN address is fixed. |

### NAT and connection tracking

A NAT rule only matches the first packet of a connection. Connection tracking remembers the translation and applies it to every later packet in both directions. [3] Turning connection tracking off therefore breaks NAT. [4]

### Limits

Inbound connections cannot reach an inside host unless a `dstnat` rule maps them. If the ISP itself uses carrier-grade NAT (`100.64.0.0/10`, B.06), your router has no public address of its own, so port forwarding at home cannot work. [5]

### How it connects

```text
B.10 srcnat masquerade ──► A.03 Step 7, C.06 NAT lab
B.10 dstnat            ──► Chapter I port forwarding and hairpin NAT
B.10 conntrack         ──► Chapter H firewall uses the same connection states
```
