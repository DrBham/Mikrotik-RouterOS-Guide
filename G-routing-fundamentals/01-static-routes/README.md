<div align="center">

# Static Routes

*Chapter G — Routing Fundamentals | Topic G.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Until now every router had one way out: the default route. This topic connects two routers, each with its own LAN, and teaches them about each other's networks with static routes. It also adds a backup route with a higher distance.

## Why This Matters

Branch offices, a separate server network, or a second router in the house all need routes to networks that are not directly connected. Static routes are the simplest way to provide them, and understanding them is the base for dynamic routing (Chapter O) and failover (Chapter N).

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.07 — Gateway and Default Route](../../B-network-fundamentals/07-gateway-and-default-route/README.md) | Routing tables, longest match, distance. |
| [B.11 — Static IP Configuration](../../B-network-fundamentals/11-static-ip-configuration/README.md) | Addressing an interface. |

## Theory

### Connected, static, and default

RouterOS creates a connected route for every IP network on an enabled interface. [1] A static route tells the router where to send traffic for a network it is not attached to: a destination (`dst-address`) and a next hop (`gateway`). [1][2] A static route without `dst-address` is a default route. [1]

### Choosing between routes

The most specific matching route is used; among routes to the same destination, the one with the lowest distance becomes active. [1] A static route has distance 1 by default; a second route with a higher distance waits as a backup and becomes active when the first one is inactive. [1][3]

### When the gateway fails

A static route stays active as long as its gateway is reachable on a connected network. `check-gateway=ping` makes the router test the gateway and deactivate the route when it stops answering. [3] With default timing this takes about 20 seconds. [3]

### Routes go both ways

Router A needs a route to router B's LAN, and router B needs a route back to router A's LAN. A missing return route is the most common static-routing mistake: the request arrives, the reply has nowhere to go.

### How it connects

```text
G.01 static routes ──► G.02 policy routing ──► Chapter N failover ──► Chapter O OSPF replaces manual routes
```

## Lab Environment

```text
LAN-A 10.1.1.0/24          Transit 10.0.12.0/30           LAN-B 10.2.2.0/24
[PC-A] ── ether2 [R1] ether1 ──────────── ether1 [R2] ether2 ── [PC-B]
          10.1.1.1     10.0.12.1        10.0.12.2      10.2.2.1
```

| Item | Value |
| --- | --- |
| R1, R2 | MikroTik CHR in GNS3, `TODO` RouterOS version |
| PC-A, PC-B | GNS3 VPCS or Linux hosts |

## Method A — CLI

### Step 1 — Addresses

R1:

```routeros
/ip/address/add address=10.0.12.1/30 interface=ether1
/ip/address/add address=10.1.1.1/24 interface=ether2
```

R2:

```routeros
/ip/address/add address=10.0.12.2/30 interface=ether1
/ip/address/add address=10.2.2.1/24 interface=ether2
```

`TODO` ![Step 1](images/cli/01-addresses.png)

### Step 2 — Show the problem

From PC-A, ping `10.2.2.10` (PC-B): it fails. R1's table has no route to `10.2.2.0/24`.

```routeros
/ip/route/print
```

`TODO` ![Step 2](images/cli/02-no-route.png)

### Step 3 — Add routes in both directions

R1:

```routeros
/ip/route/add dst-address=10.2.2.0/24 gateway=10.0.12.2
```

R2:

```routeros
/ip/route/add dst-address=10.1.1.0/24 gateway=10.0.12.1
```

[2] Ping again: it works.

`TODO` ![Step 3](images/cli/03-routes.png)

### Step 4 — Trace the path

From R1:

```routeros
/tool/traceroute 10.2.2.10 count=1
```

[4] Two hops: R2, then PC-B.

`TODO` ![Step 4](images/cli/04-traceroute.png)

### Step 5 — Add a backup route

Add a second link `10.0.21.0/30` between R1 ether3 (`.1`) and R2 ether3 (`.2`), then on R1:

```routeros
/ip/route/set [find dst-address=10.2.2.0/24] check-gateway=ping
/ip/route/add dst-address=10.2.2.0/24 gateway=10.0.21.2 distance=2
/ip/route/print where dst-address=10.2.2.0/24
```

[1][3] Disable R2 ether1 and watch the distance-2 route become active. Add the mirror routes on R2.

`TODO` ![Step 5](images/cli/05-backup-route.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| PC-A ↔ PC-B | Ping works after Step 3 | `TODO` |
| Traceroute | Via `10.0.12.2` | `TODO` |
| Primary link down | Distance-2 route active within ~20 s [3] | `TODO` |
