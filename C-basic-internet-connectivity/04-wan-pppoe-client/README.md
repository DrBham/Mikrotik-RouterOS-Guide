<div align="center">

# WAN via PPPoE Client

*Chapter C — Basic Internet Connectivity | Topic C.04*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Many fibre and DSL providers authenticate customers with PPPoE: the router logs in with a user name and password and receives its address that way. This topic configures a RouterOS PPPoE client. Because most learners have no PPPoE line to practise on, the lab also builds a small PPPoE "ISP" on a Cloud Hosted Router (CHR).

## Why This Matters

When a provider's modem is set to bridge mode, the MikroTik must dial PPPoE itself. PPPoE also changes two things you learned earlier: the WAN interface is a new virtual interface, not ether1, and its MTU is smaller than 1500.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.02 — WAN via DHCP Client](../02-wan-dhcp-client/README.md) | WAN list and testing. |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | Ethernet MTU 1500. |

## Theory

### PPP over Ethernet

PPPoE carries PPP sessions over Ethernet (RFC 2516), and RouterOS has both a client and a server. [1] A session starts with a discovery phase (PADI, PADO, PADR, PADS) and then runs PPP: LCP, authentication with PAP, CHAP, or MS-CHAP, and IPCP to get an address. [1]

### The client

| Property | Effect |
| --- | --- |
| `interface` | The Ethernet port facing the provider. [2] |
| `user`, `password` | The account from the provider. [2] |
| `add-default-route=yes` | Install a default route over the session. [1][2] |
| `use-peer-dns=yes` | Use the provider's DNS servers. [1][2] |
| `service-name` | Blank connects to any server. [2] |

The PPPoE session is its own interface (for example `pppoe-out1`), and that interface, not ether1, goes into the `WAN` list so NAT and the firewall use it. [1]

### MTU 1492

PPPoE uses 6 bytes of each Ethernet frame and PPP another 2, so a PPPoE link has an MTU of 1492 bytes. [1] The client's `max-mtu` and `max-mru` set to `auto` give 1492 on a 1500-byte Ethernet link. [2] Slow web pages while ping still works is a typical MTU symptom; MSS clamping in the PPP profiles is the fix the manual describes. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Client | MikroTik hAP lite, empty configuration + `WAN`/`LAN` lists (C.02 Steps 1, 2) |
| "ISP" | MikroTik CHR in GNS3 or VirtualBox, `TODO` RouterOS version |
| Link | hAP lite ether1 ↔ CHR ether3 (bridged to the physical NIC) |
| Account | user `alice`, password `Example-pass-1` (from the manual's example) |

> [!NOTE]
> `TODO` document how the CHR's ether3 is bridged to the laptop's second NIC or a USB Ethernet adapter, or run both routers as CHRs in GNS3.

### ISP side (CHR)

These are the manual's server commands. [1]

```routeros
/ip/pool/add name=pppoe-pool ranges=10.0.0.10-10.0.0.100
/ppp/profile/add name=subscribers local-address=10.0.0.1 remote-address=pppoe-pool
/ppp/secret/add name=alice password=Example-pass-1 profile=subscribers service=pppoe
/interface/pppoe-server/server/add service-name=internet interface=ether3 default-profile=subscribers disabled=no
```

Do not give the server interface an IP address or a DHCP server. [1]

`TODO` ![ISP side](images/cli/00-isp-server.png)
