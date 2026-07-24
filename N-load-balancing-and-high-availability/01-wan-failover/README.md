<div align="center">

# WAN Failover

*Chapter N — Load Balancing & High Availability | Topic N.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

With two internet connections, the second should take over when the first fails. This topic builds failover with route distances and `check-gateway`, then improves it by checking hosts on the internet rather than only the ISP's gateway, as the manual describes.

## Why This Matters

A second line is only useful if traffic moves to it automatically. Checking only the gateway misses the most common failure: the ISP's router answers, but the internet behind it does not.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [G.01 — Static Routes](../../G-routing-fundamentals/01-static-routes/README.md) | Distance and check-gateway. |
| [C.06 — NAT Masquerade](../../C-basic-internet-connectivity/06-nat-masquerade/README.md) | NAT on each WAN. |

## Theory

### Method 1: distance and check-gateway

The route with the lower distance is used while it is active. With `check-gateway=ping`, the router pings the gateway; if it stops answering, the route becomes inactive and the next distance takes over, in about 20 seconds with default timing. [1]

```routeros
/ip/route add distance=1 gateway=10.111.0.1 check-gateway=ping
/ip/route add distance=2 gateway=10.112.0.1 check-gateway=ping
```

[1]

### The limitation

A gateway that answers does not prove the provider has internet access. [1]

### Method 2: check a host behind each link

The manual's second method pins one public host to each link with a host route of `scope=10`, then uses that host as a recursive gateway for the default route with `target-scope=11`. If the host stops answering through that link, the default route through it fails over. [1]

```routeros
/ip/route add dst-address=8.8.8.8 scope=10 gateway=10.111.0.1
/ip/route add dst-address=8.8.4.4 scope=10 gateway=10.112.0.1
/ip/route add distance=1 gateway=8.8.8.8 target-scope=11 check-gateway=ping
/ip/route add distance=2 gateway=8.8.4.4 target-scope=11 check-gateway=ping
```

[1] Static routes have scope 30 by default, which is why the host routes need `scope=10` to be usable as gateways. [1]

### NAT and open connections

Each WAN needs its own masquerade rule. [1] Connections open at the moment of failover can break. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Router | CHR in GNS3 (or hAP lite with two uplinks), `TODO` version |
| ISP1 | ether1, gateway `10.111.0.1` (CHR) |
| ISP2 | ether2, gateway `10.112.0.1` (CHR) |
