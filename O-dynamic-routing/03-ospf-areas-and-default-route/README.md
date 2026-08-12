<div align="center">

# OSPF Areas and Default Route

*Chapter O — Dynamic Routing | Topic O.03*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic extends O.02: one router becomes the internet edge and advertises a default route into OSPF, and a branch router moves into its own area with a summarised range.

## Why This Matters

In real networks only the edge router knows the way to the internet, and branches should not carry every detail of every other site. A default route from OSPF and area summaries solve both.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [O.02 — Single-Area OSPF](../02-single-area-ospf/README.md) | The three-router lab. |

## Theory

### Default route into OSPF

`originate-default=if-installed` makes the router advertise a default route into OSPF only while it has one itself. [1]

```routeros
/routing/ospf/instance set v2inst originate-default=if-installed
```

[1]

### Redistribution

`redistribute=` injects routes from other sources, such as static or RIP routes, into OSPF. [1]

### Areas and ABRs

Every area attaches to the backbone `0.0.0.0`; a router with interfaces in two areas is an ABR. [1][2] A non-backbone area is created with its own ID:

```routeros
/routing/ospf/area add name=area1 area-id=1.1.1.1 type=default instance=v2inst
```

[1]

### Summarisation

An area range on the ABR replaces many routes from an area with one summary when advertising it to the backbone. [1]

```routeros
/routing/ospf/area/range add prefix=10.0.0.0/16 area=area1 advertise=yes cost=10
```

[1]

### Stub areas

A stub area receives no external routes; a totally stubby area (stub with `no-summaries`) also receives no inter-area routes, just a default. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Routers | O.02 lab; R1 also gets an internet uplink (`ether4`, DHCP) |
| Area 1 | R3's LAN moves to area 1 (`10.3.0.0/16`, with `10.3.3.0/24` and `10.3.4.0/24`) |

## Method A — CLI

### Step 1 — Internet on R1 and default origination

R1:

```routeros
/ip/dhcp-client/add interface=ether4
/routing/ospf/instance set v2inst originate-default=if-installed
```

[1] R2 and R3 now learn `0.0.0.0/0` from OSPF.

`TODO` ![Step 1](images/cli/01-default.png)

### Step 2 — Area 1 on R3

R3:

```routeros
/routing/ospf/area add name=area1 area-id=1.1.1.1 type=default instance=v2inst
/routing/ospf/interface-template remove [find networks=10.3.3.0/24]
/routing/ospf/interface-template add networks=10.3.0.0/16 area=area1
```

[1] Add a second LAN `10.3.4.1/24` on R3 so the summary has two routes to replace.

`TODO` ![Step 2](images/cli/02-area1.png)

### Step 3 — Summarise on the ABR

R3:

```routeros
/routing/ospf/area/range add prefix=10.3.0.0/16 area=area1 advertise=yes
```

[1] R1 and R2 now see one `10.3.0.0/16` route instead of two `/24`s.

`TODO` ![Step 3](images/cli/03-range.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| R2 default route | Learned from OSPF | `TODO` |
| R1/R2 routes to area 1 | One `10.3.0.0/16` | `TODO` |
| R1 uplink down | Default route withdrawn (`if-installed`) [1] | `TODO` |
