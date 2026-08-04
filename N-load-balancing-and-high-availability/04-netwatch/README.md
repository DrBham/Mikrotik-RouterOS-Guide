<div align="center">

# Netwatch

*Chapter N — Load Balancing & High Availability | Topic N.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Netwatch monitors hosts and runs a script when one goes down or comes back. This topic watches an internet host and a LAN server, logs their state changes, and uses a down script to switch something off.

## Why This Matters

Routes with `check-gateway` react to gateway failures only. Netwatch can react to anything you can probe, such as a website or a DNS server, and can do anything a script can do: log, disable an interface, or switch a route.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [E.06 — Scheduler and Scripts](../../E-dhcp-and-network-services/06-scheduler-and-scripts/README.md) | Scripts. |
| [N.01 — WAN Failover](../01-wan-failover/README.md) | What failover is for. |

## Theory

### Probe types

Netwatch probe types are `simple`, `icmp`, `tcp-conn`, `http-get`, `https-get`, and `dns`. [1] The default interval is 60 seconds for `simple` and 10 seconds for the others. [1] Examples from the manual: [1]

```routeros
/tool/netwatch/add host=8.8.8.8 type=icmp thr-avg=10ms
/tool/netwatch/add host=8.8.8.8 type=tcp-conn port=53
/tool/netwatch/add host=mikrotik.com type=dns dns-server=8.8.8.8 record-type=A
```

### Scripts

`up-script` and `down-script` run when the state changes. They run as the system user with only the `read`, `write`, `test`, and `reboot` policies. [1] `/tool/netwatch/print stats` shows each probe's status. [1]

### How it connects

```text
N.04 Netwatch ──► E.06 scripts ──► Chapter Q logs and alerts
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration + A.03 |
| LAN server | Laptop `192.168.88.50` |

## Method A — CLI

### Step 1 — Watch the internet

```routeros
/tool/netwatch/add host=8.8.8.8 type=icmp up-script=":log info \"internet up\"" down-script=":log warning \"internet down\""
```

[1][2]

`TODO` ![Step 1](images/cli/01-internet.png)

### Step 2 — Watch a LAN service

```routeros
/tool/netwatch/add host=192.168.88.50 type=tcp-conn port=80 down-script=":log warning \"web server down\""
```

[1] Run the web server from I.01 on the laptop.

`TODO` ![Step 2](images/cli/02-service.png)

### Step 3 — Trigger and observe

Stop the laptop's web server, wait about 10 seconds, then:

```routeros
/tool/netwatch/print stats
/log/print where message~"down|up"
```

[1][2]

`TODO` ![Step 3](images/cli/03-observe.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Web server stopped | Status down; log entry | `TODO` |
| Web server started | Status up | `TODO` |

## Reset Before Method B

```routeros
/tool/netwatch/remove [find]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Netwatch entry

**Tools → Netwatch → +**: Host, Type, and the Up/Down scripts.

`TODO` ![Step 1](images/gui/01-netwatch.png)

### Step 2 — Observe

Watch the Status column and **Log**.

`TODO` ![Step 2](images/gui/02-status.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Script does nothing | Needs a policy netwatch does not have [1] | Use commands within read/write/test/reboot. |
| Flapping up/down | Thresholds too tight | Relax `thr-*` values or interval. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/tool/netwatch/add host= type=` | Probe a host or service [1] |
| `up-script`, `down-script` | React to changes [1] |
| `print stats` | Current status [1] |

## Key Takeaways

- Netwatch watches services, not just gateways.
- Keep scripts small and log every change.

## References

1. [RouterOS Manual — Netwatch](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/netwatch/)
2. [RouterOS Manual — Logs](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/log/)

---

**Previous:** [N.03 — VRRP Gateway Redundancy](../03-vrrp-gateway-redundancy/README.md) · **Next:** Chapter O — Dynamic Routing

[Back to the main learning journal](../../README.md)
