<div align="center">

# Port Mirroring with the Switch Chip

*Chapter D — Interface & Bridge | Topic D.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Most RouterBOARD devices contain a switch chip that forwards traffic between ports in hardware. Besides speed, it offers features such as port mirroring: copying the traffic of one port to another, where a laptop with Wireshark can capture it. This topic reads the switch chip and mirrors a port.

## Why This Matters

Some problems can only be solved by looking at the packets. Mirroring lets you capture traffic of a device that cannot run Wireshark itself, such as a printer, a camera, or a phone on another port.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [D.02 — Bridge Ports and Host Table](../02-bridge-ports-and-host-table/README.md) | Bridge and hardware offload. |

## Theory

### The switch chip

RouterOS documents the built-in switch chips of RouterBOARD devices and their features: port switching, port mirroring, TX/RX limits, host table, VLAN table, rule table, port isolation, and CPU flow control. Which features exist depends on the chip model. [1] The switch is managed under `/interface/ethernet/switch`. [1]

### Port mirroring

Mirroring copies frames from a `mirror-source` port to a `mirror-target` port. [1] The manual's example:

```routeros
/interface/ethernet/switch set switch1 mirror-source=ether2 mirror-target=ether3
```

[1] The manual warns that a mirror setup involving two switch chips can create a loop and make the device unreachable. [1]

> [!WARNING]
> **Source needed:** the switch chip model of the hAP lite and whether it supports mirroring. Check `/interface/ethernet/switch/print` in Step 1 and the chip table on the [switch chip features page](https://manual.mikrotik.com/docs/bridging-and-switching/switch-chip-features/).

### Capturing on the laptop

The laptop on the target port receives copies of the source port's traffic and can capture them with Wireshark. RouterOS can also capture and stream packets itself with its packet sniffer (Chapter Q). [2]

### How it connects

```text
D.04 mirroring ──► Chapter Q packet sniffer and Wireshark
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Switch chip | `TODO` from Step 1 |
| Source | A device on ether2 (or the uplink) |
| Target | Laptop with Wireshark on ether3 |

## Method A — CLI

### Step 1 — Read the switch chip

```routeros
/interface/ethernet/switch/print
```

[1]

`TODO` ![Step 1](images/cli/01-switch.png)

### Step 2 — Mirror a port

```routeros
/interface/ethernet/switch set switch1 mirror-source=ether2 mirror-target=ether3
```

[1] Use the switch name from Step 1.

`TODO` ![Step 2](images/cli/02-mirror.png)

### Step 3 — Capture

Start Wireshark on the laptop's Ethernet adapter. Generate traffic from the ether2 device and confirm its packets appear.

> [!CAUTION]
> **Privacy:** captures contain MAC addresses, host names, and sometimes visited domains. Do not publish raw captures; screenshot only the parts needed. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 3](images/cli/03-wireshark.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Switch chip listed | Name and type | `TODO` |
| Wireshark | Frames from the source device | `TODO` |

## Reset Before Method B

```routeros
/interface/ethernet/switch set switch1 mirror-source=none mirror-target=none
```

> [!NOTE]
> `TODO` confirm the value that disables mirroring on this chip (`none`) from the CLI completion (Tab).

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Open the switch

**Switch → Switch**, open `switch1`.

`TODO` ![Step 1](images/gui/01-switch.png)

### Step 2 — Set mirror source and target

Mirror Source `ether2`, Mirror Target `ether3`. **OK**.

`TODO` ![Step 2](images/gui/02-mirror.png)

### Step 3 — Capture

Same as Method A Step 3.

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli). Disable mirroring afterwards.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| No mirrored frames | Ports on different switch chips, or chip without mirroring [1] | Check the chip table. |
| Device unreachable after mirroring | Loop through mirrored ports [1] | Remove the mirror locally or reset. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/interface/ethernet/switch/print` | Switch chips [1] |
| `mirror-source`, `mirror-target` | Copy traffic to a capture port [1] |
| Chip features vary | Check the chip table [1] |

## Key Takeaways

- The switch chip, not RouterOS, decides which hardware features exist.
- Mirroring turns any port into a capture point.

## References

1. [RouterOS Manual — Switch Chip Features](https://manual.mikrotik.com/docs/bridging-and-switching/switch-chip-features/)
2. [RouterOS Manual — Packet Sniffer](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/packet-sniffer/)

---

**Previous:** [D.03 — Spanning Tree Protocol](../03-spanning-tree-protocol/README.md) · **Next:** Chapter E — DHCP & Network Services

[Back to the main learning journal](../../README.md)
