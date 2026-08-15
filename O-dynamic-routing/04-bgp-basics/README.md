<div align="center">

# BGP Basics

*Chapter O — Dynamic Routing | Topic O.04*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

BGP is the routing protocol between autonomous systems: ISPs, data centres, and large organisations. This topic introduces its key ideas and builds an eBGP session between two CHRs in different private ASes.

## Why This Matters

A network with two ISPs and its own address space, or a small ISP connecting to an upstream, needs BGP. Even without that, understanding BGP explains how the internet chooses paths.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [O.01 — Dynamic Routing and OSPF Concepts](../01-dynamic-routing-and-ospf-concepts/README.md) | Protocol comparison. |

## Theory

### Autonomous systems

An autonomous system (AS) is a set of routers under one technical administration; its number is the ASN. Public ASNs are assigned by regional registries. [1] BGP is a path-vector protocol between ASes and uses TCP port 179. [1]

### eBGP and iBGP

eBGP connects peers in different ASes; iBGP connects peers in the same AS. [1] eBGP is normally on a directly connected link and fails by default if the peer is several hops away; iBGP is multihop by default and runs over an IGP such as OSPF. [2]

### Loop prevention and route origin

Each route carries an `AS_PATH`; a router discards a route that already contains its own ASN. [1] Routes advertised with `output.network` get origin IGP; routes from `output.redistribute` get origin incomplete. [1]

### RouterOS 7 objects

| Menu | Purpose [1][2] |
| --- | --- |
| `/routing/bgp/instance` | The local AS |
| `/routing/bgp/connection` | A neighbour; `local.role` (`ebgp`, `ibgp`, `ibgp-rr`) is mandatory |
| `/routing/bgp/session` | Live sessions; `E` means established |
| `/routing/bgp/template` | Shared settings |

RouterOS learns the remote AS from the peer's OPEN message rather than from configuration. [2] The manual's example: [2]

```routeros
/routing/bgp/instance add name=myInstance as=65531
/routing/bgp/connection add name=toR2 remote.address=192.168.1.2 instance=myInstance local.role=ebgp
```

> [!WARNING]
> BGP is not supported on SMIPS devices such as the hAP lite. [1] This lab uses CHR.

## Lab Environment

```text
AS 65531  R1 192.168.1.1 ──── 192.168.1.2 R2  AS 65532
          LAN 10.1.1.0/24                 LAN 10.2.2.0/24
```

| Item | Value |
| --- | --- |
| Routers | Two CHRs in GNS3, `TODO` version |
| ASNs | Private 65531 and 65532 (from the manual) |

## Method A — CLI

### Step 1 — Instances and connections

R1:

```routeros
/routing/bgp/instance add name=myInstance as=65531
/routing/bgp/connection add name=toR2 remote.address=192.168.1.2 instance=myInstance local.role=ebgp
```

R2:

```routeros
/routing/bgp/instance add name=myInstance as=65532
/routing/bgp/connection add name=toR1 remote.address=192.168.1.1 instance=myInstance local.role=ebgp
```

[2]

`TODO` ![Step 1](images/cli/01-bgp.png)

### Step 2 — Session

```routeros
/routing/bgp/session print
```

[2] The session shows the `E` (established) flag.

`TODO` ![Step 2](images/cli/02-session.png)

### Step 3 — Advertise the LAN

Advertise each LAN with the connection's `output.network` setting. [1]

> [!WARNING]
> **Source needed:** the exact `output.network` syntax (address list name or prefix) was not shown on the manual pages used. Check the [BGP connection CLI reference](https://manual.mikrotik.com/docs/cli-reference/routing/bgp/connection/) and record the command here.

`TODO` ![Step 3](images/cli/03-advertise.png)

### Step 4 — Check routes

```routeros
/ip/route/print where bgp
```

`TODO` confirm the filter syntax. R1 should have `10.2.2.0/24` from BGP.

`TODO` ![Step 4](images/cli/04-routes.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Session | Established | `TODO` |
| Routes | Remote LAN via BGP | `TODO` |
| LAN-to-LAN ping | Works | `TODO` |
