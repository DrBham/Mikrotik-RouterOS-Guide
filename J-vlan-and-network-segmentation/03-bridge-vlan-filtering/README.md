<div align="center">

# Bridge VLAN Filtering

*Chapter J — VLAN & Network Segmentation | Topic J.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic makes a MikroTik behave as a VLAN switch: one trunk port, two access ports in different VLANs, and a management VLAN to reach the device itself. It follows MikroTik's basic VLAN switching guide.

## Why This Matters

Bridge VLAN filtering is the recommended way to do layer-2 VLANs on RouterOS. [1] It is also the easiest place to lock yourself out of a router, so the order of the steps is part of the lesson.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [J.01 — VLAN Concepts](../01-vlan-concepts/README.md) | PVID, tagged, untagged. |
| [A.04 — RouterOS CLI Basics](../../A-mikrotik-fundamentals/04-routeros-cli-basics/README.md) | Safe mode. |

## Theory

### The manual's configuration

Trunk on ether1 (VLANs 20, 30, 99), access ports ether2 (VLAN 20) and ether3 (VLAN 30), management on VLAN 99 [2]:

```routeros
/interface/bridge add name=bridge1 frame-types=admit-only-vlan-tagged
/interface/bridge/port add bridge=bridge1 interface=ether1 frame-types=admit-only-vlan-tagged
/interface/bridge/port add bridge=bridge1 interface=ether2 pvid=20 frame-types=admit-only-untagged-and-priority-tagged
/interface/bridge/port add bridge=bridge1 interface=ether3 pvid=30 frame-types=admit-only-untagged-and-priority-tagged
/interface/bridge/vlan add bridge=bridge1 tagged=ether1 vlan-ids=20
/interface/bridge/vlan add bridge=bridge1 tagged=ether1 vlan-ids=30
/interface/bridge/vlan add bridge=bridge1 tagged=ether1,bridge1 vlan-ids=99
/interface/vlan add interface=bridge1 vlan-id=99 name=MGMT
/ip/address add address=192.168.99.1/24 interface=MGMT
/interface/bridge set bridge1 vlan-filtering=yes
```

| Line | Meaning |
| --- | --- |
| `pvid=20` with untagged frame type | Access port for VLAN 20; it is added as an untagged member automatically. [2] |
| `/interface/bridge/vlan … tagged=ether1` | VLAN carried tagged on the trunk. [2] |
| `tagged=ether1,bridge1 vlan-ids=99` | VLAN 99 also reaches the router's CPU (the bridge itself). [2][3] |
| `/interface/vlan … MGMT` + address | How you manage the device on VLAN 99. [2] |
| `vlan-filtering=yes` last | Filtering starts only after management works. [3] |

### Order matters

The manual warns not to enable VLAN filtering before management access is configured, because you can be locked out. [3] It also warns against putting several VLAN IDs on one access-port entry, which leaks VLANs to access ports. [3] Changing `vlan-filtering` also restarts all bridge ports. [4]

### Hardware

On devices whose switch chip supports it, VLAN switching runs in hardware; on multi-chip devices it only does so between ports on the same chip. [2] `TODO` record whether the hAP lite offloads it.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | hAP lite or CHR (R2 in J.02), `TODO` version |
| Starting state | Empty configuration (A.05) |
| Management | MAC WinBox until the MGMT VLAN works |

> [!WARNING]
> Use safe mode (Ctrl+X). Keep a MAC WinBox session open from a port that you will not change.

## Method A — CLI

### Step 1 — Bridge and ports

Enter the first four commands. [2]

`TODO` ![Step 1](images/cli/01-bridge-ports.png)

### Step 2 — VLAN table

Enter the three `/interface/bridge/vlan` commands, then:

```routeros
/interface/bridge/vlan/print
```

[2] The access ports appear as dynamic untagged members.

`TODO` ![Step 2](images/cli/02-vlan-table.png)

### Step 3 — Management VLAN

Enter the MGMT VLAN interface and address commands. [2]

`TODO` ![Step 3](images/cli/03-mgmt.png)

### Step 4 — Enable filtering

```routeros
/interface/bridge set bridge1 vlan-filtering=yes
```

[2] Connect to `192.168.99.1` from a device tagged in VLAN 99 on the trunk (R1 from J.02 with a VLAN 99 interface).

`TODO` ![Step 4](images/cli/04-filtering-on.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Port on ether2 | Only VLAN 20 traffic | `TODO` |
| ether2 ↔ ether3 | No direct layer-2 traffic | `TODO` |
| `192.168.99.1` | Reachable only via VLAN 99 | `TODO` |
