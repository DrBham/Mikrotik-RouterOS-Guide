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
