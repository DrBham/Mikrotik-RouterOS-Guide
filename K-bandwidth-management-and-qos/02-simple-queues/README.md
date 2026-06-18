<div align="center">

# Simple Queues

*Chapter K — Bandwidth Management & QoS | Topic K.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Simple queues limit traffic by address with a single rule. This topic limits one device, then shares a total among a parent and children, and limits by time of day.

## Why This Matters

"Limit the TV to 25 Mbps" or "give the server at least 5 Mbps" are everyday requests. Simple queues answer them without mangle rules.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [K.01 — Queue Concepts and HTB](../01-queue-concepts-and-htb/README.md) | limit-at, max-limit, FastTrack. |
| [E.01 — DHCP Static Leases and Options](../../E-dhcp-and-network-services/01-dhcp-static-leases-and-options/README.md) | A fixed address to limit. |

## Theory

### Target and direction

`target` is an address, network, or interface. "Upload" is traffic the target sends, and `max-limit=5M/25M` gives upload first, then download. [1]

### Order

The router checks simple queues from the top of the list, and a packet is handled by the first queue that matches it. [1] Specific queues must be above general ones, or added with `place-before`. [1]

### FastTrack

The default configuration's FastTrack rule bypasses simple queues. The manual disables it before using them [1]:

```routeros
/ip/firewall/filter disable [find action=fasttrack-connection]
```

### Parent and children

Children of a parent share the parent's `max-limit`; `limit-at` is only effective between queues with the same parent. [1] The manual's example: [1]

```routeros
/queue/simple add name=lan target=192.168.88.0/24 max-limit=20M/100M
/queue/simple add name=server parent=lan target=192.168.88.10/32 limit-at=5M/20M max-limit=20M/100M
/queue/simple add name=others parent=lan target=192.168.88.0/24 limit-at=15M/80M max-limit=20M/100M
```

### Time

`time=8h-18h,mon,tue,wed,thu,fri` makes a queue active only during those hours. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration + A.03 |
| Client | Laptop `192.168.88.50` (static lease) |
| Test | A browser speed test |
