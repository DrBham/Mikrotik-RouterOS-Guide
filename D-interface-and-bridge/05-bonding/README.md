<div align="center">

# Bonding

*Chapter D — Interface & Bridge | Topic D.05*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Bonding joins two or more Ethernet links into one virtual interface, for redundancy and, in some modes, more capacity. This topic connects two routers with two cables, bonds them, and pulls one cable out while traffic keeps flowing.

## Why This Matters

A single cable between a router and a core switch is a single point of failure. A bond keeps the link up when one cable, port, or transceiver fails.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [D.01 — Ethernet Interfaces and MTU](../01-ethernet-interfaces-and-mtu/README.md) | Link state of the member ports. |
| [D.03 — Spanning Tree Protocol](../03-spanning-tree-protocol/README.md) | Why two plain cables would form a loop. |

## Theory

### What a bond is

Bonding joins Ethernet-like interfaces into one virtual interface. [1] The member interfaces (slaves) must not have IP addresses of their own; the address goes on the bond. [1] The manual warns that in most modes a bond is not faster than one link for a single connection. [1]

### Modes

| Mode | Behaviour [1] |
| --- | --- |
| `balance-rr` | Default. The only mode that spreads one connection's packets over several links. |
| `active-backup` | One link active, the others wait; `primary=` chooses the active one. |
| `802.3ad` | LACP, negotiated with the switch. |
| `balance-xor` | Needs a static LAG on the switch. |
| `balance-tlb`, `balance-alb` | Balance outgoing traffic; `alb` also balances incoming IPv4 using ARP. |
| `broadcast` | Sends everything on all links. |

### Link monitoring

`link-monitoring` decides how a failed member is detected: `mii` (default, every 100 ms) watches the link state; `arp` sends ARP requests to `arp-ip-targets`; `none` turns both off. [1] The manual notes that ARP monitoring does not work correctly in `active-backup` between two directly connected routers, and should not be used with `802.3ad`. [1] With ARP monitoring, give several targets, because one failed target looks like all links failing. [1]

### How it connects

```text
D.05 bonding ──► N.03 VRRP (redundant router) ──► N.01 redundant uplink
```

## Lab Environment

| Item | Value |
| --- | --- |
| Routers | Two CHRs in GNS3 (or the hAP lite and a CHR), `TODO` versions |
| Links | ether1↔ether1 and ether2↔ether2, both removed from any bridge |
| Addresses | `172.16.0.1/24` and `172.16.0.2/24` on the bond (from the manual) |

## Method A — CLI

### Step 1 — Create the bond on both routers

```routeros
/interface/bonding/add name=bond1 slaves=ether1,ether2 mode=active-backup primary=ether1
```

[1] Router 1 address:

```routeros
/ip/address/add address=172.16.0.1/24 interface=bond1
```

Router 2: the same bond, address `172.16.0.2/24`. [1]

`TODO` ![Step 1](images/cli/01-bond.png)

### Step 2 — Check the bond

```routeros
/interface/bonding/monitor bond1
/ping 172.16.0.2
```

[1] `active-ports` shows ether1.

`TODO` ![Step 2](images/cli/02-monitor.png)

### Step 3 — Pull a cable

Run a continuous ping, then disconnect the ether1 cable. The monitor shows ether2 active and the ping continues after a short pause.

`TODO` ![Step 3](images/cli/03-failover.png)

### Step 4 — Try balance-rr

```routeros
/interface/bonding/set bond1 mode=balance-rr
```

[1] Repeat on both routers and compare with a bandwidth test (Q.06).

`TODO` ![Step 4](images/cli/04-balance-rr.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Ping over the bond | Works | `TODO` |
| ether1 unplugged | ether2 active, ping continues | `TODO` |
| balance-rr throughput | `TODO` | `TODO` |

## Reset Before Method B

```routeros
/ip/address/remove [find interface=bond1]
/interface/bonding/remove bond1
```

on both routers.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Bonding interface

**Interfaces → + → Bonding**: Slaves `ether1`, `ether2`; Mode `active backup`; Primary `ether1`.

`TODO` ![Step 1](images/gui/01-bond.png)

### Step 2 — Address and status

**IP → Addresses** on `bond1`; open the bond's **Status** tab while unplugging a cable.

`TODO` ![Step 2](images/gui/02-status.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Bond up, no traffic | Address left on a member port | Addresses belong on the bond. [1] |
| Failover not detected | ARP monitoring with direct links in active-backup | Use MII. [1] |
| LACP never forms | Switch not configured for LACP | `802.3ad` needs LACP on both ends. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/interface/bonding/add slaves= mode=` | Create a bond [1] |
| `active-backup` + `primary=` | Redundancy [1] |
| `802.3ad` | LACP to a switch [1] |
| `link-monitoring=mii` | Default detection [1] |
| `/interface/bonding/monitor` | Active and inactive ports [1] |

## Key Takeaways

- Bonding is mainly about redundancy; one connection rarely gets faster.
- Put the address on the bond, never on its members.

## References

1. [RouterOS Manual — Bonding](https://manual.mikrotik.com/docs/high-availability-solutions/bonding/)

---

**Previous:** [D.04 — Port Mirroring with the Switch Chip](../04-port-mirroring-with-the-switch-chip/README.md) · **Next:** [Chapter E — DHCP & Network Services](../../E-dhcp-and-network-services/01-dhcp-static-leases-and-options/README.md)

[Back to the main learning journal](../../README.md)
