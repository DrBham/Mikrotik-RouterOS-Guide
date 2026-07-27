<div align="center">

# PCC Load Balancing

*Chapter N — Load Balancing & High Availability | Topic N.02*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

N.01 used the second line only when the first failed. Per Connection Classifier (PCC) uses both at once by spreading connections between them, while keeping each connection on one line. This topic builds the manual's two-link PCC setup.

## Why This Matters

Two modest connections used together give more total bandwidth to a busy office. Keeping each connection on one line matters, because websites and banks break when packets of one session arrive from two addresses.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [G.02 — Policy Routing](../../G-routing-fundamentals/02-policy-routing/README.md) | Routing tables and marks. |
| [K.04 — Queue Tree with Mangle](../../K-bandwidth-management-and-qos/04-queue-tree-with-mangle/README.md) | Mangle connection marks. |
| [N.01 — WAN Failover](../01-wan-failover/README.md) | Two WAN links. |

## Theory

### What PCC does

PCC is a firewall matcher that divides connections into groups so that all packets of one connection stay in the same group. [1] Its syntax is `ValuesToHash:Denominator/Remainder`, for example `both-addresses:2/0` and `both-addresses:2/1`. [1] Options to hash include `both-addresses`, `src-address`, `both-addresses-and-ports`, and others. [1]

### From match to route

PCC only matches. The steps are: mark new connections with PCC, turn connection marks into routing marks, and have one routing table per provider. [1] Marking happens in `prerouting` so the routing mark is set before the routing decision. [1] Connected networks are excluded first so local traffic is not sent to an ISP. [1]

### The manual's rules (core)

```routeros
/routing/table add name=ISP1_table fib
/routing/table add name=ISP2_table fib
/ip/firewall/mangle add chain=prerouting action=mark-connection connection-mark=no-mark connection-state=new dst-address-type=!local in-interface=ether_LAN per-connection-classifier=both-addresses:2/0 new-connection-mark=ISP1
/ip/firewall/mangle add chain=prerouting action=mark-connection connection-mark=no-mark connection-state=new dst-address-type=!local in-interface=ether_LAN per-connection-classifier=both-addresses:2/1 new-connection-mark=ISP2
/ip/firewall/mangle add chain=prerouting action=mark-routing connection-mark=ISP1 in-interface=ether_LAN new-routing-mark=ISP1_table
/ip/firewall/mangle add chain=prerouting action=mark-routing connection-mark=ISP2 in-interface=ether_LAN new-routing-mark=ISP2_table
/ip/route add gateway=10.10.4.1 routing-table=ISP1_table check-gateway=ping
/ip/route add gateway=10.10.5.1 routing-table=ISP2_table check-gateway=ping
/ip/route add gateway=10.10.4.1 distance=1 check-gateway=ping
/ip/route add gateway=10.10.5.1 distance=2 check-gateway=ping
```

[1] The full example also marks the router's own connections (input/output chains) and excludes connected networks with an address list; see the manual. [1]

### Limits

PCC does not work with HotSpot. [1] FastTrack works only with the `main` table (G.02), so marked connections must not be FastTracked.

## Lab Environment

| Item | Value |
| --- | --- |
| Router | CHR in GNS3, `TODO` version |
| ISP1 | `ether_ISP1` `10.10.4.100/24`, gateway `10.10.4.1` |
| ISP2 | `ether_ISP2` `10.10.5.100/24`, gateway `10.10.5.1` |
| LAN | `ether_LAN` `192.168.100.1/24` |

Rename interfaces with `/interface/set ether1 name=ether_ISP1` etc. to match the manual.
