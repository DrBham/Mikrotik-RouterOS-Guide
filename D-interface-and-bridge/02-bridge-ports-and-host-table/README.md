<div align="center">

# Bridge Ports and Host Table

*Chapter D — Interface & Bridge | Topic D.02*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

C.05 created a bridge to build the LAN. This topic looks inside it: the bridge's own settings, its ports, the host table it learns, and the fast paths that make it quick. It also shows where bridged traffic meets the firewall.

## Why This Matters

Most LAN problems on a MikroTik involve the bridge: a port that is not a member, a port in two places, or a firewall rule that never sees bridged traffic. Reading the host table answers "which port is that device on?" without walking to the switch.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | MAC learning. |
| [C.05 — LAN Bridge, IP, and DHCP Server](../../C-basic-internet-connectivity/05-lan-bridge-ip-and-dhcp-server/README.md) | The bridge you built. |

## Theory

### What a bridge does

MAC bridges join Ethernet-like networks so that hosts on separate segments act as one LAN. Bridges are transparent: they do not appear in traceroute. [1] To combine ports, RouterOS creates a bridge interface and adds the ports to it. [1]

### Bridge defaults

| Setting | Default [1][2] |
| --- | --- |
| `protocol-mode` | `rstp` |
| `priority` | `0x8000` (32768) |
| `auto-mac` | `yes`: the bridge takes its MAC from a port |
| `ageing-time` | 5 minutes: how long a learned host is kept |
| `fast-forward` | `yes` for new bridges |

### The host table

The bridge keeps a table of learned MAC addresses and the port each was seen on, under `/interface/bridge/host`. [1] Entries that are not seen again are removed after `ageing-time`. [1]

### Fast paths

Fast Forward is a faster path for a bridge with exactly two running ports, with hardware offloading, VLAN filtering, and DHCP snooping off. It also disables MAC learning. [1] Fast Forward is disabled when hardware offloading is enabled, because the switch chip then forwards the traffic itself. [1]

> [!WARNING]
> Changing `fast-forward`, `vlan-filtering`, `protocol-mode`, or `igmp-snooping` temporarily disables all bridge ports. [1]

### Bridged traffic and the IP firewall

Traffic between two ports of the same bridge only passes the bridge firewall. To filter it with `/ip/firewall/filter` as well, set `use-ip-firewall=yes` in `/interface/bridge/settings`. [1]

### How it connects

```text
D.02 bridge ──► D.03 STP between bridges ──► Chapter J VLAN filtering on the bridge
            └─► Chapter H firewall: bridged vs routed traffic
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | C.05 or default configuration |
| Devices | Laptop on ether2, a second device (phone over Wi-Fi or a PC on ether3) |
