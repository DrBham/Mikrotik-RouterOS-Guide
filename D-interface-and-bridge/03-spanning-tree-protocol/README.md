<div align="center">

# Spanning Tree Protocol

*Chapter D — Interface & Bridge | Topic D.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A loop between bridges or switches makes frames circulate forever and can take a network down within seconds. The Spanning Tree Protocol (STP) and its faster version RSTP prevent this by blocking redundant paths. This topic explains how a root bridge and port roles are chosen, then creates a loop on purpose and watches RSTP block it.

## Why This Matters

Someone will eventually plug both ends of a cable into the same switch. With RSTP on (the RouterOS default), that is a non-event; without it, the whole LAN stops. Knowing how to set bridge priorities also decides which device becomes the centre of a multi-switch network.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [D.02 — Bridge Ports and Host Table](../02-bridge-ports-and-host-table/README.md) | Bridge settings and ports. |

## Theory

### Why loops are a problem

Bridges exchange BPDUs to discover redundant paths and block them so that no loop exists. [1][2] The bridge with the lowest bridge ID becomes the root bridge. [1]

### Electing the root

| Choice | Decided by [2] |
| --- | --- |
| Root bridge | Lowest bridge priority, then lowest bridge MAC |
| Root port (on every other bridge) | Lowest path cost to the root, then lowest bridge ID, then lowest port ID |

Bridge priority should be set in steps of 4096 (`0x1000`), from 0 to 61440. RouterOS accepts any value up to 65535, but the manual warns about compatibility. [2] The manual recommends setting each bridge's priority manually rather than relying on MAC addresses. [2]

### Port settings

| Setting | Default | Meaning [2] |
| --- | --- | --- |
| Port `priority` | `0x80` | Tie-breaker between ports, in steps of 16 |
| `path-cost` | Depends on speed | Cost toward the root |
| `edge` | `auto` | Edge ports face end devices, not other bridges |
| `point-to-point` | | `yes` speeds up convergence on direct links |

### RSTP is the default

New RouterOS bridges use `protocol-mode=rstp`. [1] MSTP adds separate trees per group of VLANs (Chapter J). [2] For STP to work through bridge filter rules, frames to `01:80:C2:00:00:00` must be allowed. [2]

### How it connects

```text
D.03 STP ──► Chapter J VLANs (MSTP per VLAN group) ──► Chapter N redundancy
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration (ether2–ether4 in `bridge`) |
| Extra | One short Ethernet cable to connect ether3 to ether4 |

> [!WARNING]
> Do this only with RSTP enabled (check Step 1). With STP disabled, the loop floods the LAN.
