<div align="center">

# Traffic Flow

*Chapter Q — Network Monitoring & Troubleshooting | Topic Q.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

SNMP counts how much traffic passes an interface; Traffic Flow says who talked to whom. RouterOS exports flow records in NetFlow and IPFIX formats to a collector. This topic enables Traffic Flow and sends it to a collector on the laptop.

## Why This Matters

"Who is using all the bandwidth?" is answered by flows, not by interface counters. Torch (C.07) shows this live on the router; Traffic Flow keeps a history on another machine.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.07 — Verify and Troubleshoot Connectivity](../../C-basic-internet-connectivity/07-verify-and-troubleshoot-connectivity/README.md) | Torch, the live equivalent. |
| [Q.03 — SNMP Monitoring](../03-snmp-monitoring/README.md) | Monitoring basics. |

## Theory

### What is exported

RouterOS exports flows as NetFlow v1, v5, v9, or IPFIX. [1] Only traffic that passes through the CPU is counted; hardware-offloaded bridged traffic and mirrored ports are not seen. [1]

### The manual's configuration

```routeros
/ip/traffic-flow/set enabled=yes
/ip/traffic-flow/target/add dst-address=192.168.0.2 port=2055 version=9
/ip/traffic-flow/monitor duration=3
```

[1] Packet sampling reduces load on busy routers. [1]

### Privacy

Flow records list every address and port your devices contact. Treat the collector's data as private.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, A.03 |
| Collector | Laptop `192.168.88.50`, UDP 2055, NetFlow v9 collector (`TODO` tool) |

## Method A — CLI

### Step 1 — Enable and add a target

```routeros
/ip/traffic-flow/set enabled=yes
/ip/traffic-flow/target/add dst-address=192.168.88.50 port=2055 version=9
```

[1]

`TODO` ![Step 1](images/cli/01-enable.png)

### Step 2 — Monitor the export

```routeros
/ip/traffic-flow/monitor duration=3
```

[1]

`TODO` ![Step 2](images/cli/02-monitor.png)

### Step 3 — Read flows on the collector

Browse a few websites from a phone, then look at the collector's top talkers.

> [!CAUTION]
> **Privacy:** flows show visited addresses and public IPs. Cover them. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 3](images/cli/03-collector.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Monitor | Flows being exported | `TODO` |
| Collector | Records from the phone's address | `TODO` |

## Reset Before Method B

```routeros
/ip/traffic-flow/target/remove [find]
/ip/traffic-flow/set enabled=no
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Traffic Flow

**IP → Traffic Flow**: Enabled. **Targets → +**: Address `192.168.88.50`, Port `2055`, Version `9`.

`TODO` ![Step 1](images/gui/01-traffic-flow.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| No flows at the collector | Laptop firewall blocks UDP 2055, or wrong version | Match version and port. [1] |
| Bridged LAN traffic missing | Hardware offload bypasses the CPU [1] | Expected. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/ip/traffic-flow/set enabled=yes` | Start exporting [1] |
| `target/add dst-address= port= version=` | Where to send [1] |
| NetFlow v1/v5/v9, IPFIX | Formats [1] |
| CPU traffic only | Offloaded traffic is invisible [1] |

## Key Takeaways

- Flows answer "who talked to whom"; counters only say "how much".
- Flow data is personal data; protect the collector.

## References

1. [RouterOS Manual — Traffic Flow](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/traffic-flow/)

---

**Previous:** [Q.03 — SNMP Monitoring](../03-snmp-monitoring/README.md) · **Next:** Q.05 — Packet Sniffer and Wireshark

[Back to the main learning journal](../../README.md)
