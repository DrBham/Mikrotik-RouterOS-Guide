<div align="center">

# VRRP Gateway Redundancy

*Chapter N — Load Balancing & High Availability | Topic N.03*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Clients have one default gateway. If that router fails, the LAN is offline even when a second router is ready. The Virtual Router Redundancy Protocol (VRRP) lets two routers share one virtual gateway address, so the backup takes over automatically.

## Why This Matters

N.01 and N.02 made the internet link redundant. VRRP makes the router itself redundant, which matters for any network that cannot afford an outage while a router is replaced or rebooted.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.07 — Gateway and Default Route](../../B-network-fundamentals/07-gateway-and-default-route/README.md) | The default gateway. |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | Virtual MAC addresses. |

## Theory

### How VRRP works

RouterOS supports VRRPv2 (RFC 3768) and VRRPv3 (RFC 5798, IPv4 and IPv6). [1] Routers in one group send advertisements to `224.0.0.18` (IP protocol 112). [1] The group shares a virtual MAC, `00:00:5E:00:01:<VRID>` for IPv4. [1]

| Setting | Default [1] |
| --- | --- |
| `vrid` | 1 |
| `priority` | 100 (1–254) |
| `interval` | 1 s |
| `preemption-mode` | yes: a higher-priority router takes over when it returns |

The router with the highest priority becomes master; on a tie, the one with the higher primary IP wins. [1]

### Addresses

Each router needs a real address on the parent interface; without it, VRRP reports "No IPv4 address!". [1] The virtual address goes on the VRRP interface, as a `/32` if it is in the same subnet as another interface. [1] The manual's example: [1]

```routeros
/interface/vrrp add name=vrrp1 interface=ether5
/ip/address add address=192.168.1.2/24 interface=ether5
/ip/address add address=192.168.1.1/32 interface=vrrp1
```

### Connection tracking sync

`sync-connection-tracking=yes` copies connection state to the backup so NAT sessions survive a failover; it uses UDP 8275. [1]

## Lab Environment

```text
          ┌── R1 ether5 192.168.1.2  (priority 200) ──┐
LAN ──────┤        virtual 192.168.1.1                 ├── upstream
          └── R2 ether5 192.168.1.3  (priority 100) ──┘
```

| Item | Value |
| --- | --- |
| Routers | Two CHRs in GNS3, `TODO` version |
| Client | VPCS with gateway `192.168.1.1` |

## Method A — CLI

### Step 1 — Real addresses

R1: `/ip/address add address=192.168.1.2/24 interface=ether5`
R2: `/ip/address add address=192.168.1.3/24 interface=ether5`

[1]

### Step 2 — VRRP interfaces

R1:

```routeros
/interface/vrrp add name=vrrp1 interface=ether5 priority=200
/ip/address add address=192.168.1.1/32 interface=vrrp1
```

R2:

```routeros
/interface/vrrp add name=vrrp1 interface=ether5
/ip/address add address=192.168.1.1/32 interface=vrrp1
```

[1]

`TODO` ![Step 2](images/cli/02-vrrp.png)

### Step 3 — Check roles

```routeros
/interface/vrrp print
```

[1] R1 is master, R2 backup.

`TODO` ![Step 3](images/cli/03-roles.png)

### Step 4 — Fail over

Run a continuous ping from the client to `192.168.1.1`, then shut down R1. R2 becomes master; the ping loses only a few replies. Start R1 again; it takes back the master role (preemption).

`TODO` ![Step 4](images/cli/04-failover.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Normal | R1 master | `TODO` |
| R1 down | R2 master; gateway keeps answering | `TODO` |
| R1 back | R1 master again [1] | `TODO` |

## Reset Before Method B

Remove the VRRP interfaces and their addresses on both routers.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — VRRP interface

**Interfaces → + → VRRP**: Interface `ether5`, VRID `1`, Priority (200 on R1).

`TODO` ![Step 1](images/gui/01-vrrp.png)

### Step 2 — Virtual address

**IP → Addresses → +**: `192.168.1.1/32` on `vrrp1`.

`TODO` ![Step 2](images/gui/02-address.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| "No IPv4 address!" | No real address on the parent | Step 1. [1] |
| Both routers master | Advertisements not seen (filtered or different VRID) | Allow protocol 112; match VRID. [1] |
| Traffic to the virtual address dropped | `rp-filter=strict` | Use `loose`. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/interface/vrrp add interface=` | VRRP group member [1] |
| Virtual address on vrrp interface | Shared gateway [1] |
| Highest priority = master | Default 100 [1] |
| `sync-connection-tracking` | Keep sessions on failover [1] |

## Key Takeaways

- VRRP makes the gateway address survive a router failure.
- Each router still needs its own real address.

## References

1. [RouterOS Manual — VRRP](https://manual.mikrotik.com/docs/high-availability-solutions/vrrp/)

---

**Previous:** [N.02 — PCC Load Balancing](../02-pcc-load-balancing/README.md) · **Next:** N.04 — Netwatch

[Back to the main learning journal](../../README.md)
