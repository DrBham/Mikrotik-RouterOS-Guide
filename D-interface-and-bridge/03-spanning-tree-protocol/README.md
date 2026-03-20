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

## Method A — CLI

### Step 1 — Check the protocol and priority

```routeros
/interface/bridge/print detail
/interface/bridge/monitor bridge
```

[1][2] `protocol-mode` should be `rstp`. The monitor shows whether this bridge is the root.

`TODO` ![Step 1](images/cli/01-rstp.png)

### Step 2 — Create a loop

Connect a cable from ether3 to ether4. Both are ports of the same bridge, so frames could circle back.

```routeros
/interface/bridge/port/monitor [find]
```

[1] One of the two ports should become an alternate or backup port and stop forwarding.

`TODO` ![Step 2](images/cli/02-loop-blocked.png)

### Step 3 — Influence the choice with port priority

```routeros
/interface/bridge/port/set [find interface=ether4] priority=0x70
/interface/bridge/port/monitor [find]
```

A lower port priority wins the tie. [2] Watch which port now forwards.

`TODO` ![Step 3](images/cli/03-port-priority.png)

### Step 4 — Set the bridge priority

```routeros
/interface/bridge/set bridge priority=0x1000
```

[2] On a single router this keeps it the root; in a multi-switch network it makes this device the root on purpose.

`TODO` ![Step 4](images/cli/04-bridge-priority.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Loop cable connected | One port not forwarding | `TODO` |
| LAN during the loop | Still works (laptop pings 192.168.88.1) | `TODO` |
| After priority change | The other port blocks | `TODO` |

## Reset Before Method B

```routeros
/interface/bridge/port/set [find interface=ether4] priority=0x80
/interface/bridge/set bridge priority=0x8000
```

Unplug the loop cable.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Check STP settings

**Bridge → bridge → STP** tab: Protocol Mode `RSTP`, Priority.

`TODO` ![Step 1](images/gui/01-stp-settings.png)

### Step 2 — Create the loop and watch the ports

Connect ether3 to ether4. **Bridge → Ports**: add the Role and Status columns.

`TODO` ![Step 2](images/gui/02-port-roles.png)

### Step 3 — Change the port priority

Open ether4 in **Bridge → Ports → STP** and set Priority `70`.

`TODO` ![Step 3](images/gui/03-port-priority.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli). Unplug the loop cable afterwards.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| LAN freezes when a loop cable is connected | `protocol-mode=none` | Set `rstp`. [1] |
| Unexpected device becomes root | Priorities left at default; lowest MAC wins [2] | Set priorities manually. [2] |
| STP does not work through a bridge filter | BPDUs blocked | Allow `01:80:C2:00:00:00`. [2] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| Root bridge | Lowest priority, then lowest MAC [2] |
| Priority steps | 4096 (`0x1000`) [2] |
| Port priority | `0x80` default, lower wins [2] |
| `protocol-mode=rstp` | Default on new bridges [1] |

## Key Takeaways

- RSTP turns an accidental loop into a blocked port instead of an outage.
- Choose the root on purpose with bridge priority.
- Never disable STP on a bridge with more than one path.

## References

1. [RouterOS Manual — Bridging and Switching](https://manual.mikrotik.com/docs/bridging-and-switching/)
2. [RouterOS Manual — Spanning Tree Protocol](https://manual.mikrotik.com/docs/bridging-and-switching/user-guides/spanning-tree-protocol/)

---

**Previous:** [D.02 — Bridge Ports and Host Table](../02-bridge-ports-and-host-table/README.md) · **Next:** D.04 — Port Mirroring with the Switch Chip

[Back to the main learning journal](../../README.md)
