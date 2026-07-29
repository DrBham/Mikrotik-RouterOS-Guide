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

## Method A — CLI

### Step 1 — Addresses and tables

```routeros
/ip/address add address=10.10.4.100/24 interface=ether_ISP1
/ip/address add address=10.10.5.100/24 interface=ether_ISP2
/ip/address add address=192.168.100.1/24 interface=ether_LAN
/routing/table add name=ISP1_table fib
/routing/table add name=ISP2_table fib
```

[1]

`TODO` ![Step 1](images/cli/01-tables.png)

### Step 2 — Exclude connected networks

```routeros
/ip/firewall/address-list add address=10.10.4.0/24 list=ConnectedNetworks
/ip/firewall/address-list add address=10.10.5.0/24 list=ConnectedNetworks
/ip/firewall/address-list add address=192.168.100.0/24 list=ConnectedNetworks
/ip/firewall/mangle add chain=prerouting action=accept dst-address-list=ConnectedNetworks
/ip/firewall/mangle add chain=output action=accept dst-address-list=ConnectedNetworks
```

[1]

`TODO` ![Step 2](images/cli/02-connected.png)

### Step 3 — Classify and route

Enter the four mangle rules and four routes from the theory section, plus the input/output rules from the manual's full example. [1]

`TODO` ![Step 3](images/cli/03-pcc.png)

### Step 4 — NAT

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade out-interface=ether_ISP1
/ip/firewall/nat add chain=srcnat action=masquerade out-interface=ether_ISP2
```

[1]

### Step 5 — Watch the split

Generate many connections from LAN hosts (several browser tabs or downloads), then:

```routeros
/ip/firewall/mangle/print stats
/ip/firewall/connection/print
```

[1]

`TODO` ![Step 5](images/cli/05-stats.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Mark counters | Roughly even between ISP1 and ISP2 | `TODO` |
| One ISP down | All new connections use the other (check-gateway) | `TODO` |

## Reset Before Method B

Remove the mangle rules, address list, routing tables and their routes, and the two NAT rules.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Tables and routes

**Routing → Tables**, **IP → Routes** as in Method A.

`TODO` ![Step 1](images/gui/01-tables-routes.png)

### Step 2 — Mangle

**IP → Firewall → Mangle → +**: Advanced tab → Per Connection Classifier `both addresses`, `2/0` and `2/1`.

`TODO` ![Step 2](images/gui/02-mangle.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| LAN cannot reach the router or local subnets | Connected networks not excluded | Step 2. [1] |
| All traffic on one ISP | FastTrack, or rules not in prerouting | Exclude marked traffic from FastTrack; check chains. [1] |
| HotSpot users broken | PCC is incompatible with HotSpot [1] | Do not combine them. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `per-connection-classifier=both-addresses:2/0` | Group connections [1] |
| mark-connection → mark-routing | Turn groups into tables [1] |
| One table + default route per ISP | Where each group goes [1] |
| ConnectedNetworks accept | Keep local traffic local [1] |

## Key Takeaways

- PCC balances connections, not packets, so sessions stay on one line.
- Always exclude local networks before marking.

## References

1. [RouterOS Manual — Per Connection Classifier](https://manual.mikrotik.com/docs/high-availability-solutions/load-balancing/per-connection-classifier/)

---

**Previous:** [N.01 — WAN Failover](../01-wan-failover/README.md) · **Next:** N.03 — VRRP Gateway Redundancy

[Back to the main learning journal](../../README.md)
