<div align="center">

# Policy Routing

*Chapter G — Routing Fundamentals | Topic G.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Normal routing looks only at the destination. Policy routing also looks at who is sending, which interface the packet came in on, or a mark set by the firewall, and can send that traffic through a different routing table. This topic sends one LAN device out of a second uplink while everyone else uses the first.

## Why This Matters

With two internet connections, you may want a server or a VoIP phone on the stable line and everything else on the fast one. Policy routing is how RouterOS makes that choice, and the same tools are used for load balancing in Chapter N.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [G.01 — Static Routes](../01-static-routes/README.md) | Routes and distance. |
| [C.06 — NAT Masquerade](../../C-basic-internet-connectivity/06-nat-masquerade/README.md) | NAT per uplink. |

## Theory

### Routing tables

RouterOS starts with one table, `main`. Additional tables are created under `/routing/table` with the `fib` flag so they can be used for forwarding. [1] Routes are placed in a table with `routing-table=`. [1]

### Two ways to choose a table

| Tool | Matches on | Example [1][2] |
| --- | --- | --- |
| Routing rule | Source, destination, incoming interface | `/routing/rule/add src-address=192.168.88.50/32 action=lookup table=to-wan2` |
| Mangle `mark-routing` | Anything the firewall can match | Mark the connection, then set `new-routing-mark` |

`action=lookup` falls back to `main` if the table has no matching route; `lookup-only-in-table` does not. [1] By default, mangle routing marks take priority over routing rules. [2]

### FastTrack

FastTrack only works with the `main` table. [2] Traffic marked for another table must not be FastTracked (Chapter H, K).

### NAT per uplink

Each uplink needs its own masquerade rule so that replies come back through the same link the request left from. [1]

### How it connects

```text
G.02 tables + rules ──► Chapter N PCC load balancing ──► Chapter M routing traffic into tunnels
```

## Lab Environment

| Item | Value |
| --- | --- |
| Router | MikroTik hAP lite or CHR, `TODO` version |
| WAN1 | ether1, DHCP (C.02) |
| WAN2 | ether2 removed from the bridge, gateway `10.20.0.1` (second CHR or phone tethering) |
| Special device | `192.168.88.50` (static lease, E.01) |

> [!NOTE]
> `TODO` document how the second uplink is provided in your lab.

## Method A — CLI

The commands follow the manual's multi-WAN example. [1]

### Step 1 — A second routing table

```routeros
/routing/table/add name=to-wan2 fib
```

[1]

`TODO` ![Step 1](images/cli/01-table.png)

### Step 2 — Default route in that table

```routeros
/ip/route/add dst-address=0.0.0.0/0 gateway=10.20.0.1 routing-table=to-wan2 check-gateway=ping
```

[1]

`TODO` ![Step 2](images/cli/02-route.png)

### Step 3 — Rule for one device

```routeros
/routing/rule/add src-address=192.168.88.50/32 action=lookup table=to-wan2
```

[1]

`TODO` ![Step 3](images/cli/03-rule.png)

### Step 4 — NAT on the second uplink

```routeros
/ip/firewall/nat/add chain=srcnat out-interface=ether2 action=masquerade
```

[1]

`TODO` ![Step 4](images/cli/04-nat.png)

### Step 5 — Test

From `192.168.88.50` and from another LAN device, open a "what is my IP" page or run a traceroute. The first hop differs.

> [!CAUTION]
> **Privacy:** cover public addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 5](images/cli/05-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `.50` first hop | WAN2 gateway | `TODO` |
| Other devices | WAN1 gateway | `TODO` |
| WAN2 down | `.50` falls back to `main` (because of `action=lookup`) [1] | `TODO` |
