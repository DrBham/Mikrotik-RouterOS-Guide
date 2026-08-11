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
