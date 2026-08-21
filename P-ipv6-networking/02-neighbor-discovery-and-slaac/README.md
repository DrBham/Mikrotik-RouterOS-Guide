<div align="center">

# Neighbor Discovery and SLAAC

*Chapter P — IPv6 Networking | Topic P.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

IPv6 hosts can configure their own addresses from router advertisements, without DHCP. This topic explains Neighbor Discovery (ND) and stateless address autoconfiguration (SLAAC), then gives the LAN an IPv6 subnet so the laptop configures itself.

## Why This Matters

ND replaces ARP and the gateway part of DHCP. Understanding it explains why IPv6 clients get addresses even without a DHCP server, and what the router must advertise for them to work.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [P.01 — IPv6 Addressing](../01-ipv6-addressing/README.md) | Prefixes, link-local, `/64`. |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | What ND replaces. |

## Theory

### Neighbor Discovery

ND defines Router Solicitation and Advertisement, Neighbor Solicitation and Advertisement, and Redirect messages. [1] It combines the functions of IPv4's ARP, ICMP Router Discovery, and ICMP Redirect. [1] Address resolution works by multicasting a Neighbor Solicitation instead of broadcasting. [1] RFC 8415 likewise describes ND as replacing and enhancing ARP. [2]

### SLAAC

Routers advertise prefixes; hosts generate an interface identifier and combine the two to form an address. [3] Hosts send Router Solicitations to ask for advertisements. [3] A link-local address is the `fe80::` prefix plus the interface identifier. [3] Duplicate Address Detection must be performed on every unicast address before it is used. [3]

### RouterOS settings

| Setting | Where | Meaning |
| --- | --- | --- |
| `advertise=yes` | `/ipv6/address` | Advertise this prefix for SLAAC; on by default for a `/64`. [4] |
| `ra-interval`, `ra-lifetime` | `/ipv6/nd` | How often to advertise; lifetime 0 means "not a default router". [5] |
| `advertise-dns` | `/ipv6/nd` | Advertise the router's DNS servers to hosts. [5] |
| `managed-address-configuration`, `other-configuration` | `/ipv6/nd` | Tell hosts to use DHCPv6 for addresses or other settings. [5] |

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration |
| Prefix | ULA `fd00:88::/64` (works without ISP IPv6) |
| Client | Laptop on ether2 |

> [!NOTE]
> A ULA prefix gives working IPv6 inside the LAN without internet IPv6. P.03 adds a global prefix from the ISP. `TODO` choose a random ULA Global ID as RFC 4193 recommends; `fd00:88::` is a simple lab example.

## Method A — CLI

### Step 1 — Look at link-local addresses

```routeros
/ipv6/address/print
```

[4]

`TODO` ![Step 1](images/cli/01-link-local.png)

### Step 2 — Add a LAN prefix

```routeros
/ipv6/address/add address=fd00:88::1/64 interface=bridge advertise=yes
```

[4]

`TODO` ![Step 2](images/cli/02-prefix.png)

### Step 3 — Advertise DNS

```routeros
/ipv6/nd/print
/ipv6/nd/set [find interface=all] advertise-dns=yes
```

[5]

`TODO` ![Step 3](images/cli/03-nd.png)

### Step 4 — Laptop

```powershell
ipconfig
ping fd00:88::1
```

The laptop shows a `fd00:88::` address it built itself, and the router's link-local address as the IPv6 default gateway.

> [!CAUTION]
> **Privacy:** cover any global IPv6 addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 4](images/cli/04-laptop.png)

### Step 5 — The neighbour table

```routeros
/ipv6/neighbor/print
```

`TODO` confirm the menu name. The laptop appears with its link-local and ULA addresses.

`TODO` ![Step 5](images/cli/05-neighbors.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Laptop IPv6 | `fd00:88::…` from SLAAC [3] | `TODO` |
| Laptop gateway | Router `fe80::` address | `TODO` |
| Ping `fd00:88::1` | Replies | `TODO` |
