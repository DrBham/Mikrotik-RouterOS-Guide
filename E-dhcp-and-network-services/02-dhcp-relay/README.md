<div align="center">

# DHCP Relay

*Chapter E — DHCP & Network Services | Topic E.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A DHCP DISCOVER is a broadcast, and routers do not forward broadcasts (B.04). A DHCP relay solves this: it listens on a LAN, forwards client requests to a DHCP server on another network, and passes the answers back. This topic builds a central DHCP server with a relay in front of it.

## Why This Matters

Larger networks keep one DHCP server for many subnets or branches instead of one server per router. The relay is what makes that possible, and it is the same idea behind DHCP in many enterprise and ISP networks.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | Broadcast DISCOVER. |
| [E.01 — DHCP Static Leases and Options](../01-dhcp-static-leases-and-options/README.md) | DHCP server parts. |

## Theory

### How the relay works

The relay forwards requests to every DHCP server in its list and writes its own `local-address` into the request (the `giaddr` field). [1] The server uses that address to know which network the client is on, and it must be configured with `relay=` set to that address. [1] New relays are created disabled, so add `disabled=no`. [1]

### The manual's example

Relay router, two LANs, one central server `192.168.0.1` [1]:

```routeros
/ip/dhcp-relay/add name=Local1-Relay interface=Local1 dhcp-server=192.168.0.1 local-address=192.168.1.1 disabled=no
/ip/dhcp-relay/add name=Local2-Relay interface=Local2 dhcp-server=192.168.0.1 local-address=192.168.2.1 disabled=no
```

Server side, one pool per relayed network [1]:

```routeros
/ip/pool/add name=Local1-Pool ranges=192.168.1.11-192.168.1.100
/ip/dhcp-server/add interface=To-DHCP-Relay relay=192.168.1.1 address-pool=Local1-Pool name=DHCP-1
```

The relay can also add option 82 (relay information) with `add-relay-info=yes`. [1]

### How it connects

```text
Client ── broadcast ──► Relay (hAP lite) ── unicast ──► Central DHCP server (CHR)
```

## Lab Environment

| Item | Value |
| --- | --- |
| Relay | hAP lite, LAN `192.168.1.1/24` on ether3 (laptop) |
| Server | CHR in GNS3/VirtualBox, `192.168.0.1/24`, reachable from the hAP lite |
| Link | hAP lite ether4 `192.168.0.2/24` ↔ CHR |
| RouterOS versions | `TODO` |

> [!NOTE]
> `TODO` draw the topology and document how the CHR is connected to the hAP lite (second NIC or USB adapter bridged in VirtualBox/GNS3).

## Method A — CLI

### Step 1 — Addresses

On the hAP lite (remove ether3 and ether4 from the bridge first, as in B.11):

```routeros
/ip/address/add address=192.168.1.1/24 interface=ether3
/ip/address/add address=192.168.0.2/24 interface=ether4
```

On the CHR:

```routeros
/ip/address/add address=192.168.0.1/24 interface=ether1
/ip/route/add dst-address=192.168.1.0/24 gateway=192.168.0.2
```

`TODO` ![Step 1](images/cli/01-addresses.png)

### Step 2 — DHCP server on the CHR

```routeros
/ip/pool/add name=Local1-Pool ranges=192.168.1.11-192.168.1.100
/ip/dhcp-server/network/add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=192.168.1.1
/ip/dhcp-server/add interface=ether1 relay=192.168.1.1 address-pool=Local1-Pool name=DHCP-1
```

[1][2]

`TODO` ![Step 2](images/cli/02-server.png)

### Step 3 — Relay on the hAP lite

```routeros
/ip/dhcp-relay/add name=Local1-Relay interface=ether3 dhcp-server=192.168.0.1 local-address=192.168.1.1 disabled=no
```

[1]

`TODO` ![Step 3](images/cli/03-relay.png)

### Step 4 — Test

Renew the laptop on ether3. On the CHR:

```routeros
/ip/dhcp-server/lease/print
```

`TODO` ![Step 4](images/cli/04-lease.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Laptop address | From `192.168.1.11–100` | `TODO` |
| Lease shown on | CHR, not hAP lite | `TODO` |
