<div align="center">

# Single-Area OSPF

*Chapter O — Dynamic Routing | Topic O.02*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic replaces the static routes of G.01 with OSPF. Three routers in a triangle share their LANs through the backbone area, and when one link fails, traffic takes the other way automatically.

## Why This Matters

The triangle is the smallest network where dynamic routing clearly wins: with static routes, a link failure needs manual changes; with OSPF it does not.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [O.01 — Dynamic Routing and OSPF Concepts](../01-dynamic-routing-and-ospf-concepts/README.md) | Areas, router ID, neighbours. |
| [G.01 — Static Routes](../../G-routing-fundamentals/01-static-routes/README.md) | The lab being replaced. |

## Theory

### RouterOS 7 OSPF in three objects

| Object | Purpose [1] |
| --- | --- |
| Instance | One OSPF process, with its version and router ID |
| Area | An area in that instance; the backbone is `0.0.0.0` |
| Interface template | Which networks or interfaces run OSPF, and in which area |

The manual's basic example: [1]

```routeros
/routing/ospf/instance add name=v2inst version=2 router-id=1.2.3.4
/routing/ospf/area add name=backbone_v2 area-id=0.0.0.0 instance=v2inst
/routing/ospf/interface-template add networks=192.168.0.0/24 area=backbone_v2
```

`networks=` enables OSPF on interfaces whose addresses fall in that range. [1]

## Lab Environment

```text
           R1 (1.1.1.1)  LAN 10.1.1.0/24
          /            \
 10.0.12.0/30        10.0.13.0/30
        /                \
 R2 (2.2.2.2) ──10.0.23.0/30── R3 (3.3.3.3)
 LAN 10.2.2.0/24               LAN 10.3.3.0/24
```

| Item | Value |
| --- | --- |
| Routers | Three CHRs in GNS3, `TODO` version |
| Addresses | `.1` on the lower-numbered router of each link |

## Method A — CLI

### Step 1 — Addresses

Configure the links and LANs on each router as in the diagram, with no static routes.

`TODO` ![Step 1](images/cli/01-addresses.png)

### Step 2 — OSPF on each router

R1 (change the router ID on R2 and R3):

```routeros
/routing/ospf/instance add name=v2inst version=2 router-id=1.1.1.1
/routing/ospf/area add name=backbone_v2 area-id=0.0.0.0 instance=v2inst
/routing/ospf/interface-template add networks=10.0.0.0/16 area=backbone_v2
/routing/ospf/interface-template add networks=10.1.1.0/24 area=backbone_v2
```

[1] The `10.0.0.0/16` template covers all three transit links; the second template adds the LAN (use `10.2.2.0/24` and `10.3.3.0/24` on R2 and R3).

`TODO` ![Step 2](images/cli/02-ospf.png)

### Step 3 — Neighbours and routes

```routeros
/routing/ospf/neighbor/print
/ip/route/print where ospf
```

> [!NOTE]
> `TODO` confirm these two status commands on your RouterOS version; they were not in the manual pages used for this topic.

Each router should have two neighbours and routes to the two remote LANs.

`TODO` ![Step 3](images/cli/03-neighbours.png)

### Step 4 — Break a link

Disable the R1–R2 link. After the dead interval (40 s by default [2]), R1 reaches `10.2.2.0/24` via R3.

```routeros
/tool/traceroute 10.2.2.1 count=1
```

`TODO` ![Step 4](images/cli/04-reroute.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Neighbours | Two per router | `TODO` |
| LAN routes | Learned by OSPF | `TODO` |
| R1–R2 down | Path via R3 | `TODO` |
