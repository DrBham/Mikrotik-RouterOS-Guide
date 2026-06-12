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
