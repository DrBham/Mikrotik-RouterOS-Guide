<div align="center">

# WAN via DHCP Client

*Chapter C — Basic Internet Connectivity | Topic C.02*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Starting from an empty configuration, this topic connects the router itself to the internet with a DHCP client on the WAN interface. At the end, the router can ping `8.8.8.8` and `google.com`. The LAN comes in C.05.

## Why This Matters

Most home and office uplinks hand out addresses by DHCP: a home router, a phone hotspot, most cable and fibre modems in router mode. This is the most common WAN type and the one the default configuration uses on ether1. [1]

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.01 — Lab Topology and Planning](../01-lab-topology-and-planning/README.md) | Interface plan and lists. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | DORA, leases, options. |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | Wireless station steps, if your WAN is wlan1. |

## Theory

### What the DHCP client does

The DHCP client on the WAN interface asks the upstream network for an address. When it gets a lease, RouterOS adds the address to the interface as a dynamic address and removes it when the lease is lost or the client is disabled. [2]

| Property | Default | Effect |
| --- | --- | --- |
| `add-default-route` | `yes` | Adds a default route through the received gateway (option 3), or the classless routes (option 121) if the server sends them. [2] |
| `default-route-distance` | `1` | Distance of that route (B.07). [2] |
| `use-peer-dns` | `yes` | Received DNS servers become the router's dynamic DNS servers. [2][3] |
| `use-peer-ntp` | | Received NTP servers go to the NTP client. [2] |

The status moves through `searching...` and `requesting...` to `bound`. [4]

### The router's own DNS

The router resolves names for itself using its DNS servers; static `servers` are tried before dynamic ones. [3] Serving DNS to the LAN (`allow-remote-requests`) is a separate setting used in C.05. [3]

### Marking the WAN

The interface that faces the internet goes into the `WAN` interface list, so C.06's NAT rule and Chapter H's firewall can refer to it by role. [5][6]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| WAN | `TODO` ether1 (cable) or wlan1 (station, see A.03) |
| Laptop | ether2, WinBox by MAC |
| Starting state | Empty configuration |

## Method A — CLI

### Step 1 — Start from an empty configuration

```routeros
/system/reset-configuration no-defaults=yes skip-backup=yes
```

[7] After the reboot, connect with WinBox by MAC address and open **New Terminal**. [8]

> [!CAUTION]
> **Privacy:** cover the MAC address in the Neighbors list. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-empty-config.png)

### Step 2 — Create the interface lists

```routeros
/interface/list add name=WAN
/interface/list add name=LAN
```

[6]

`TODO` ![Step 2](images/cli/02-interface-lists.png)

### Step 3 — Prepare the WAN interface

**ether1:** connect the upstream cable. Nothing else to do.

**wlan1:** repeat A.03 Steps 3 and 5 (security profile and station mode). In an empty configuration wlan1 is not in any bridge, so A.03 Step 4 is not needed. [9]

`TODO` ![Step 3](images/cli/03-wan-interface.png)

### Step 4 — Add the DHCP client

```routeros
/ip/dhcp-client/add interface=ether1
/ip/dhcp-client/print detail
```

[2] Use `interface=wlan1` if that is your WAN. Wait for `status=bound`. [2]

`TODO` ![Step 4](images/cli/04-dhcp-client-bound.png)

### Step 5 — Put the WAN interface in the WAN list

```routeros
/interface/list/member add list=WAN interface=ether1
```

[6]

`TODO` ![Step 5](images/cli/05-wan-list.png)

### Step 6 — Check address, route, and DNS

```routeros
/ip/address/print
/ip/route/print
/ip/dns/print
```

Expect a dynamic address on the WAN, a default route `0.0.0.0/0` with the DHCP flag and distance 1, and dynamic DNS servers. [2][3][10]

`TODO` ![Step 6](images/cli/06-address-route-dns.png)

### Step 7 — Test from the router

```routeros
/ping 8.8.8.8 count=3
/ping google.com count=3
```

[11][1]

`TODO` ![Step 7](images/cli/07-ping.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| DHCP client | `bound` [2] | `TODO` |
| Default route | `0.0.0.0/0`, distance 1 [2] | `TODO` |
| DNS | Dynamic servers listed [3] | `TODO` |
| Ping 8.8.8.8 and google.com | Replies | `TODO` |
