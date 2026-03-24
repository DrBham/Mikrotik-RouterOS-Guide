<div align="center">

# DHCP Static Leases and Options

*Chapter E — DHCP & Network Services | Topic E.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

C.05 gave every client a random address from the pool. This topic gives one device a fixed address through a static lease, limits a client's speed from the lease, and sends extra DHCP options.

## Why This Matters

Printers, servers, cameras, and port-forwarding targets (Chapter I) need addresses that do not change. A static lease keeps the device on DHCP, so it still receives the right gateway and DNS, while its address stays the same. It is safer than typing a static address on the device, which the DHCP server would not know about.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | Leases and options. |
| [C.05 — LAN Bridge, IP, and DHCP Server](../../C-basic-internet-connectivity/05-lan-bridge-ip-and-dhcp-server/README.md) | A running DHCP server. |

## Theory

### Dynamic and static leases

Dynamic leases disappear when they expire or are released; static leases stay. [1] A client with a static lease always receives its address. The static address does not have to be inside the pool, and static addresses are not conflict-checked. [1] Keeping static addresses outside the pool avoids handing the same address to another device.

### Lease time

The default lease time is 30 minutes, and clients renew at 50% and rebind at 87.5% of it. [2][3]

### Options

DHCP options (B.09) can be added beyond gateway and DNS. Where an option is defined decides which one wins: RADIUS first, then the lease, then the server, then the network entry. [1] The server only sends an option the client asks for, unless the option has `force=yes`. [1]

### Per-client rate limit

A static lease can carry a `rate-limit`, which limits that client's speed. [1] (Chapter K covers queues in general.)

### How it connects

```text
E.01 fixed address ──► Chapter I port forwarding target ──► Chapter K per-device limits
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration (or C.05) + A.03 internet |
| Client | Laptop on ether2, DHCP |
| Fixed address | `192.168.88.50` |
