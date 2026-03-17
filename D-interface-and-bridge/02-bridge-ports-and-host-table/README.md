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

## Method A — CLI

### Step 1 — Inspect the bridge

```routeros
/interface/bridge/print detail
/interface/bridge/monitor bridge-lan
```

[1][2] Use your bridge's name (`bridge` in the default configuration).

`TODO` ![Step 1](images/cli/01-bridge-monitor.png)

### Step 2 — Inspect the ports

```routeros
/interface/bridge/port/print
/interface/bridge/port/monitor [find]
```

[1]

`TODO` ![Step 2](images/cli/02-ports.png)

### Step 3 — Read the host table

```routeros
/interface/bridge/host/print
```

[1] Find the laptop's MAC on ether2. Move the laptop cable to ether3, wait, and print again.

> [!CAUTION]
> **Privacy:** cover MAC addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 3](images/cli/03-host-table.png)

### Step 4 — Check the fast-path counters

```routeros
/interface/bridge/settings/print
```

[1]

`TODO` ![Step 4](images/cli/04-settings.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Laptop MAC | Learned on ether2, then on ether3 after moving [1] | `TODO` |
| Port roles | All ports listed with their status | `TODO` |

## Reset Before Method B

Method A only read information. Move the laptop back to ether2.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Bridge settings

**Bridge → Bridge** tab, open the bridge: General and STP tabs.

`TODO` ![Step 1](images/gui/01-bridge.png)

### Step 2 — Ports

**Bridge → Ports**.

`TODO` ![Step 2](images/gui/02-ports.png)

### Step 3 — Hosts

**Bridge → Hosts**. Move the laptop cable and watch the entry change port.

`TODO` ![Step 3](images/gui/03-hosts.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Device on a port gets no DHCP address | Port not a bridge member | `/interface/bridge/port/print`. [1] |
| Firewall rule does not match LAN-to-LAN traffic | Bridged traffic skips the IP firewall | `use-ip-firewall=yes`, or use the bridge filter. [1] |
| All LAN ports dropped briefly | A bridge setting that restarts ports was changed [1] | Expected; avoid changing it remotely. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/interface/bridge/host/print` | Which MAC is on which port [1] |
| `ageing-time` 5 min | How long hosts are remembered [1] |
| Fast Forward | Two-port fast path; off with hardware offload [1] |
| `use-ip-firewall=yes` | Send bridged traffic through the IP firewall [1] |

## Key Takeaways

- The host table tells you where a device is plugged in.
- Bridged traffic does not pass the IP firewall unless you ask for it.
- Some bridge settings restart every port; change them locally.

## References

1. [RouterOS Manual — Bridging and Switching](https://manual.mikrotik.com/docs/bridging-and-switching/)
2. [RouterOS Manual — Spanning Tree Protocol](https://manual.mikrotik.com/docs/bridging-and-switching/user-guides/spanning-tree-protocol/)

---

**Previous:** [D.01 — Ethernet Interfaces and MTU](../01-ethernet-interfaces-and-mtu/README.md) · **Next:** D.03 — Spanning Tree Protocol

[Back to the main learning journal](../../README.md)
