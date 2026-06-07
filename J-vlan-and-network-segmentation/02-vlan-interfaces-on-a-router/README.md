<div align="center">

# VLAN Interfaces on a Router

*Chapter J — VLAN & Network Segmentation | Topic J.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The simplest VLAN setup puts VLAN interfaces on one router port, which connects as a trunk to a managed switch. Each VLAN interface gets its own address, so the router routes between VLANs. This topic builds two VLANs on one port.

## Why This Matters

This "router on a stick" design is how a small office with one MikroTik and one managed switch separates its networks. It also shows that a VLAN interface on RouterOS behaves like any other interface: it can have addresses, DHCP, and firewall rules.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [J.01 — VLAN Concepts](../01-vlan-concepts/README.md) | Tags, trunks, access ports. |
| [C.05 — LAN Bridge, IP, and DHCP Server](../../C-basic-internet-connectivity/05-lan-bridge-ip-and-dhcp-server/README.md) | DHCP per network. |

## Theory

### VLAN interfaces

`/interface/vlan` creates a virtual interface with a `vlan-id` on a parent `interface`. Frames sent through it leave the parent tagged; tagged frames with that ID arriving on the parent appear on the VLAN interface. [1] The manual's example:

```routeros
/interface/vlan add name=VLAN2 vlan-id=2 interface=ether1 disabled=no
/ip/address add address=10.10.20.1/24 interface=VLAN2
```

[1]

### Inter-VLAN routing

With one VLAN interface and one address per VLAN on the trunk port, the router routes between the VLANs like between any two networks. The manual's example uses VLANs 2, 3, and 4 on ether1 with `10.10.20.1`, `10.10.30.1`, and `10.10.40.1`. [1]

### Lab without a managed switch

A second MikroTik device or a CHR can act as the "switch" side and carry the VLANs; a Windows laptop can also tag frames only if its network driver supports VLAN IDs. This lab uses two CHRs in GNS3 so no special hardware is needed.

### How it connects

```text
J.02 VLAN on a router port ──► J.03 the switch side with bridge VLAN filtering ──► J.04 firewall between VLANs
```

## Lab Environment

```text
[PC-20] ── R2 (switch role) ether2 access VLAN 20
[PC-30] ── R2 (switch role) ether3 access VLAN 30
R2 ether1 trunk (20, 30) ─────── ether1 R1 (router role)
```

| Item | Value |
| --- | --- |
| R1, R2 | MikroTik CHR in GNS3, `TODO` version |
| R2 switch config | From J.03 (or any VLAN-aware switch) |

## Method A — CLI

On R1.

### Step 1 — VLAN interfaces

```routeros
/interface/vlan add name=VLAN20 vlan-id=20 interface=ether1 disabled=no
/interface/vlan add name=VLAN30 vlan-id=30 interface=ether1 disabled=no
```

[1]

`TODO` ![Step 1](images/cli/01-vlan-interfaces.png)

### Step 2 — Addresses

```routeros
/ip/address add address=10.10.20.1/24 interface=VLAN20
/ip/address add address=10.10.30.1/24 interface=VLAN30
```

[1]

`TODO` ![Step 2](images/cli/02-addresses.png)

### Step 3 — DHCP per VLAN

```routeros
/ip/dhcp-server/setup
```

Run it once for `VLAN20` and once for `VLAN30`. [2]

`TODO` ![Step 3](images/cli/03-dhcp.png)

### Step 4 — Test

PC-20 gets `10.10.20.x`, PC-30 gets `10.10.30.x`, and each can ping the other through R1.

`TODO` ![Step 4](images/cli/04-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `/interface/vlan/print` | Two VLANs on ether1, running | `TODO` |
| PC addresses | One per subnet | `TODO` |
| PC-20 → PC-30 | Routed via R1 | `TODO` |
