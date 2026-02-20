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
