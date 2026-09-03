<div align="center">

# Resources, Profiler, and Graphing

*Chapter Q — Network Monitoring & Troubleshooting | Topic Q.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

When a router feels slow, the question is whether it is the router or the network. This topic reads CPU and memory, finds which process uses the CPU with the profiler, and turns on graphs for interface traffic and resources over time.

## Why This Matters

The hAP lite has one 650 MHz core and 32 MB of RAM. [4] Firewall rules, queues, or a packet capture can saturate it, and only the resource tools show it.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.08 — Upgrading RouterOS and Packages](../../A-mikrotik-fundamentals/08-upgrading-routeros-and-packages/README.md) | `/system/resource/print` basics. |
| [K.01 — Queue Concepts and HTB](../../K-bandwidth-management-and-qos/01-queue-concepts-and-htb/README.md) | What loads the CPU. |

## Theory

### Resources

`/system/resource/print` shows the version, uptime, CPU load, memory, and storage; `/system/resource/monitor` shows CPU and memory live; `/system/resource/cpu/print` gives a per-core load summary. [1]

### Profiler

`/tool/profile` shows which parts of RouterOS use the CPU, grouped by classifiers such as `console`, `wireless`, `bridging`, and `firewall`. [2] `cpu=total` is the default; `cpu=all` or a core number shows individual cores. [2]

### Graphing

Nothing is graphed by default; rules for interfaces, resources, and queues are added under `/tool/graphing`. [3] Graphs are served by the router's web service at `/graphs/`, with averages kept for a day, week, month, and year. [3] The default `allow-address` is `0.0.0.0/0`, so always restrict it. [3]

```routeros
/tool/graphing/interface/add interface=ether1 allow-address=192.168.88.10/32
/tool/graphing/resource/add allow-address=192.168.88.10/32
```

[3]

> [!NOTE]
> A.06 disabled the `www` service. Graphs need it, so this lab re-enables it for the LAN only (Step 4).

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, A.03 + A.06 |
| Laptop | `192.168.88.50` |

## Method A — CLI

### Step 1 — Baseline

```routeros
/system/resource/print
/system/resource/cpu/print
```

[1]

`TODO` ![Step 1](images/cli/01-resource.png)

### Step 2 — Load the CPU and watch

Start a large download on the laptop, then:

```routeros
/system/resource/monitor duration=10
```

[1]

`TODO` ![Step 2](images/cli/02-monitor.png)

### Step 3 — Find the cause

```routeros
/tool/profile cpu=total duration=5
```

[2] Note which classifiers use the most CPU during the download.

`TODO` ![Step 3](images/cli/03-profile.png)

### Step 4 — Turn on graphs

```routeros
/ip/service/enable www
/ip/service/set www available-from=192.168.88.0/24
/tool/graphing/interface/add interface=ether1 allow-address=192.168.88.50/32
/tool/graphing/resource/add allow-address=192.168.88.50/32
```

[3][5] After a few minutes, open `http://192.168.88.1/graphs/` on the laptop.

`TODO` ![Step 4](images/cli/04-graphs.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| CPU during download | `TODO` % | `TODO` |
| Top profiler entry | `TODO` | `TODO` |
| `/graphs/` from laptop | Interface and resource graphs | `TODO` |
| `/graphs/` from another address | Refused (`allow-address`) | `TODO` |

## Reset Before Method B

```routeros
/tool/graphing/interface/remove [find]
/tool/graphing/resource/remove [find]
/ip/service/disable www
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Resources

**System → Resources**, and the CPU graph in the WinBox status bar.

`TODO` ![Step 1](images/gui/01-resources.png)

### Step 2 — Profile

**Tools → Profile → Start**.

`TODO` ![Step 2](images/gui/02-profile.png)

### Step 3 — Graphing

**Tools → Graphing → Interface Rules → +** and **Resource Rules → +**, Allow Address `192.168.88.50/32`.

`TODO` ![Step 3](images/gui/03-graphing.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| CPU at 100% during transfers | Firewall, queues, or no FastTrack | Profile; review Chapters H and K. [2] |
| `/graphs/` does not open | `www` disabled or `available-from` excludes the laptop | Step 4. [5] |
| Graphs visible to everyone | `allow-address` left at `0.0.0.0/0` | Restrict it. [3] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/system/resource/print`, `monitor`, `cpu/print` | Load and memory [1] |
| `/tool/profile` | CPU by function [2] |
| `/tool/graphing/…/add allow-address=` | Graphs over time [3] |
| `/graphs/` | Where to view them [3] |

## Key Takeaways

- Check the CPU before blaming the network.
- Restrict graph access; it shows your traffic.

## References

1. [RouterOS Manual — Resource](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/resource/)
2. [RouterOS Manual — Profiler](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/profiler/)
3. [RouterOS Manual — Graphing](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/graphing/)
4. [MikroTik — hAP lite (RB941-2nD)](https://mikrotik.com/product/RB941-2nD)
5. [RouterOS Manual — Securing your router](https://manual.mikrotik.com/docs/getting-started/securing-your-router/)

---

**Previous:** [Q.01 — Logs and Remote Syslog](../01-logs-and-remote-syslog/README.md) · **Next:** Q.03 — SNMP Monitoring

[Back to the main learning journal](../../README.md)
