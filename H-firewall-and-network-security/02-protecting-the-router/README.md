<div align="center">

# Protecting the Router

*Chapter H — Firewall & Network Security | Topic H.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Starting from the Chapter C router (empty configuration plus WAN, LAN, and NAT), this topic writes the `input` chain that protects the router itself, following the rules in MikroTik's firewall guide.

## Why This Matters

The Chapter C build has no input filter at all, so every service on the router is reachable from the WAN. This topic closes that gap and shows why the order of five short rules matters.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.01 — Firewall Concepts and Packet Flow](../01-firewall-concepts-and-packet-flow/README.md) | Chains, order, states. |
| [C.06 — NAT Masquerade](../../C-basic-internet-connectivity/06-nat-masquerade/README.md) | The router being protected. |
| [A.04 — RouterOS CLI Basics](../../A-mikrotik-fundamentals/04-routeros-cli-basics/README.md) | Safe mode. |

## Theory

### The manual's input rules

MikroTik's firewall guide protects the router with an address list of management addresses and five `input` rules [1]:

```routeros
/ip/firewall/address-list add list=allowed_to_router address=192.168.88.2-192.168.88.254
/ip/firewall/filter add chain=input action=accept connection-state=established,related,untracked comment="Accept established, related, untracked"
/ip/firewall/filter add chain=input action=drop connection-state=invalid comment="Drop invalid"
/ip/firewall/filter add chain=input action=accept src-address-list=allowed_to_router in-interface-list=LAN comment="Accept from the management addresses"
/ip/firewall/filter add chain=input action=accept protocol=icmp comment="Accept ICMP"
/ip/firewall/filter add chain=input action=drop comment="Drop everything else"
```

| Rule | Why |
| --- | --- |
| Accept established/related/untracked | Replies to the router's own connections (DNS, NTP, updates) [2] |
| Drop invalid | Packets that belong to nothing [2] |
| Accept from management addresses on LAN | Your access to WinBox, SSH, DNS |
| Accept ICMP | Ping and the ICMP errors that make the network work [1] |
| Drop everything else | Default deny |

### Lockout risk

The last rule drops everything not accepted above, including your own WinBox session if your address is not in the list or your interface is not in `LAN`. The manual warns about this and suggests safe mode. [1]

### DNS from the WAN

The final drop also covers DNS from the WAN, so the separate rules from C.06 become redundant but harmless. [3]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Chapter C build (C.02 + C.05 + C.06), lists `WAN` and `LAN` |
| Laptop | ether2, `192.168.88.x` |
