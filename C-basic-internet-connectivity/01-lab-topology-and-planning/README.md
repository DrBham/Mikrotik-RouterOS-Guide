<div align="center">

# Lab Topology and Planning

*Chapter C — Basic Internet Connectivity | Topic C.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [The plan](#the-chapter-c-plan) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Chapter C builds an internet gateway by hand, starting from a router with no configuration at all. Each topic adds one piece and explains it with the theory from Chapter B. This first topic plans the lab: which interface is WAN, which is LAN, which addresses to use, and in what order to build things.

## Why This Matters

In A.03 the default configuration did most of the work. That is convenient, but it hides how a gateway is built. Writing the plan first, as a table of interfaces and addresses, is also how real network changes are done: it shows conflicts before they reach the router.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.05 — Default Configuration and Reset](../../A-mikrotik-fundamentals/05-default-configuration-and-reset/README.md) | Starting from an empty configuration. |
| [Chapter B](../../B-network-fundamentals/01-what-is-a-computer-network/README.md) | All of the theory used here. |

## Theory

### What a SOHO gateway needs

The official First Time Configuration guide walks through the same pieces this chapter builds: an internet connection on the WAN side (DHCP, static IP, or PPPoE), a connectivity check, and NAT for the LAN. [1] The default configuration of an AP-type router adds the LAN side: a bridge with `192.168.88.1/24`, a DHCP server, DNS, and a firewall. [2]

| Piece | Chapter B theory | Topic |
| --- | --- | --- |
| WAN address and default route | B.07, B.09 | C.02, C.03, C.04 |
| DNS for the router and the LAN | B.08 | C.02, C.03 |
| LAN bridge, address, DHCP server | B.03, B.04, B.09 | C.05 |
| NAT from LAN to WAN | B.06, B.10 | C.06 |
| Testing and troubleshooting | B.02, B.07 | C.07 |

### WAN and LAN interface lists

RouterOS interface lists group interfaces so that firewall and NAT rules can refer to a role such as `WAN` instead of a port name. [3] The official firewall guide creates the two lists like this:

```routeros
/interface/list add name=WAN
/interface/list add name=LAN
/interface/list/member add list=WAN interface=ether1
/interface/list/member add list=LAN interface=bridge
```

[4] Adding a bridge to a list is not the same as adding all its ports. [5] When the WAN changes, for example from ether1 to wlan1 or to a PPPoE interface, only the list membership changes; the NAT and firewall rules stay the same. [6]

### Why start from an empty configuration

After `/system/reset-configuration no-defaults=yes`, nothing is configured. [7] Every entry you see afterwards is one you added, so you know exactly what each command did. Access is through a MAC WinBox connection until the LAN has an IP address. [8]

## The Chapter C plan

```text
Internet
   │
[ Upstream: home router / phone hotspot / ISP ]
   │  WAN: ether1 (cable)  or  wlan1 (Wi-Fi client, as in A.03)
[ hAP lite ]
   │  LAN: bridge-lan = ether2, ether3, ether4
[ Laptop ]
```

| Item | Value | Topic |
| --- | --- | --- |
| WAN interface | `ether1`, or `wlan1` in station mode | C.02 |
| WAN address | From DHCP (C.02), static (C.03), or PPPoE (C.04) | C.02–C.04 |
| LAN bridge | `bridge-lan` with ether2–ether4 | C.05 |
| LAN address | `192.168.88.1/24` | C.05 |
| DHCP pool | `192.168.88.10–192.168.88.254` (excludes the router) | C.05 |
| DNS for clients | `192.168.88.1` (router resolver) | C.05 |
| NAT | `srcnat` `masquerade` out of the `WAN` list | C.06 |

> [!IMPORTANT]
> The upstream network must not use `192.168.88.0/24` (B.05). If it does, choose another LAN subnet such as `192.168.77.0/24` and use it in every topic.

### Build order

1. Reset to an empty configuration and connect by MAC.
2. WAN: get an address and a default route; ping `8.8.8.8` from the router.
3. DNS: ping `google.com` from the router.
4. LAN: bridge, address, DHCP server; the laptop gets an address.
5. NAT: the laptop reaches the internet.
6. Verify and troubleshoot layer by layer.

Each step can be tested before the next one, so a fault is always in the last thing you changed.

### How it connects

```text
C.01 plan ──► C.02–C.04 WAN ──► C.05 LAN ──► C.06 NAT ──► C.07 verify ──► Chapter H adds the firewall
```

## Quick Recap

| Item | Meaning |
| --- | --- |
| WAN / LAN lists | Roles that NAT and firewall rules refer to [3][4] |
| Empty start | `reset-configuration no-defaults=yes`, then MAC login [7][8] |
| Build order | WAN, DNS, LAN, NAT, verify [1] |
| LAN plan | `192.168.88.1/24`, pool excluding the router [9] |

## Key Takeaways

- Write the interface and address plan before touching the router.
- Rules that use the `WAN` and `LAN` lists survive a change of WAN interface.
- Build and test one layer at a time.

## References

1. [RouterOS Manual — First Time Configuration](https://manual.mikrotik.com/docs/getting-started/first-time-configuration)
2. [RouterOS Manual — Default configurations](https://manual.mikrotik.com/docs/getting-started/configuration-management/default-configurations/)
3. [RouterOS CLI Reference — interface/list](https://manual.mikrotik.com/docs/cli-reference/interface/list/)
4. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)
5. [RouterOS CLI Reference — interface/list/member](https://manual.mikrotik.com/docs/cli-reference/interface/list/member/)
6. [RouterOS Manual — PPPoE](https://manual.mikrotik.com/docs/virtual-private-networks/pppoe/)
7. [RouterOS Manual — Configuration Management](https://manual.mikrotik.com/docs/getting-started/configuration-management/)
8. [RouterOS Manual — MAC server](https://manual.mikrotik.com/docs/management-tools/mac-server/)
9. [RouterOS Manual — DHCP Server](https://manual.mikrotik.com/docs/network-management/dhcp/server/)

---

**Previous:** [B.11 — Static IP Configuration](../../B-network-fundamentals/11-static-ip-configuration/README.md) · **Next:** C.02 — WAN via DHCP Client

[Back to the main learning journal](../../README.md)
