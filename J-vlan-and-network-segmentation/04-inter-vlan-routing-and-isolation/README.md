<div align="center">

# Inter-VLAN Routing and Isolation

*Chapter J — VLAN & Network Segmentation | Topic J.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

J.02 and J.03 built VLANs, and the router routes between them freely. This topic adds the policy: both VLANs reach the internet, the staff VLAN may reach the guest VLAN's printer, and guests reach nothing internal.

## Why This Matters

Separation is only real when the firewall enforces it. Without rules, VLANs are just tidier subnets that can still talk to each other through the router.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [J.02 — VLAN Interfaces on a Router](../02-vlan-interfaces-on-a-router/README.md) | The VLANs to separate. |
| [H.03 — Protecting the LAN](../../H-firewall-and-network-security/03-protecting-the-lan/README.md) | Forward chain and first-match order. |

## Theory

### Interface lists as roles

Interface lists group interfaces for the firewall. [1] Putting each VLAN interface in a role list (`STAFF`, `GUEST`) lets rules refer to roles instead of interface names. [2]

### Forward rules

Traffic between VLANs passes the `forward` chain, where rules are checked top to bottom and the first match decides. [1] The policy is written as: accept established/related first (H.03), then the specific allows, then a drop for everything else between internal networks.

### Example policy

| From → To | Action |
| --- | --- |
| Any VLAN → internet (WAN) | Accept |
| STAFF → GUEST printer `10.10.30.50` | Accept |
| GUEST → STAFF | Drop |
| Any → any else internal | Drop |

### How it connects

```text
J.04 VLAN policy ──► Chapter K per-VLAN queues ──► Chapter L HotSpot on the guest VLAN
```

## Lab Environment

| Item | Value |
| --- | --- |
| Router | R1 from J.02 (VLAN20 = STAFF, VLAN30 = GUEST), with a WAN for internet |
| RouterOS version | `TODO` |
| Hosts | PC-20 (staff), PC-30 (guest, also plays the printer at `10.10.30.50`) |

## Method A — CLI

### Step 1 — Role lists

```routeros
/interface/list add name=STAFF
/interface/list add name=GUEST
/interface/list/member add list=STAFF interface=VLAN20
/interface/list/member add list=GUEST interface=VLAN30
```

[1][2]

`TODO` ![Step 1](images/cli/01-lists.png)

### Step 2 — Forward policy

```routeros
/ip/firewall/filter add chain=forward action=accept connection-state=established,related,untracked comment="Accept established, related, untracked"
/ip/firewall/filter add chain=forward action=drop connection-state=invalid comment="Drop invalid"
/ip/firewall/filter add chain=forward action=accept in-interface-list=STAFF dst-address=10.10.30.50 comment="Staff to printer"
/ip/firewall/filter add chain=forward action=accept out-interface-list=WAN comment="All VLANs to internet"
/ip/firewall/filter add chain=forward action=drop comment="Drop everything else"
```

[1] The first two rules are the manual's standard start. [1]

> [!WARNING]
> The last rule also drops new connections from the WAN into the VLANs, which is the intended default. Add port-forward exceptions (Chapter I) above it if needed.

`TODO` ![Step 2](images/cli/02-policy.png)

### Step 3 — Test the policy

| From | Test | Expected |
| --- | --- | --- |
| PC-20 | `ping 10.10.30.50` | Works |
| PC-30 | `ping 10.10.20.x` | Fails |
| Both | `ping 8.8.8.8` | Works |

```routeros
/ip/firewall/filter/print stats where chain=forward
```

`TODO` ![Step 3](images/cli/03-tests.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Staff → printer | Allowed | `TODO` |
| Guest → staff | Dropped | `TODO` |
| Both → internet | Allowed | `TODO` |
