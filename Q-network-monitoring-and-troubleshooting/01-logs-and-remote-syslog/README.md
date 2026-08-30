<div align="center">

# Logs and Remote Syslog

*Chapter Q — Network Monitoring & Troubleshooting | Topic Q.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The log is the first place to look when something changed and nobody knows why. This topic reads and filters the RouterOS log, keeps a separate debug buffer for one feature, saves logs to disk, and sends them to a syslog server on the laptop.

## Why This Matters

The default log lives in memory and disappears on reboot, which is exactly when you need it most. Keeping logs on disk or on another machine turns "it rebooted and stopped working" into something you can investigate.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [E.04 — NTP, Clock, and Identity](../../E-dhcp-and-network-services/04-ntp-clock-and-identity/README.md) | Log timestamps must be correct. |
| [E.06 — Scheduler and Scripts](../../E-dhcp-and-network-services/06-scheduler-and-scripts/README.md) | Scripts write to the log. |

## Theory

### Default logging

By default, `info`, `error`, and `warning` messages go to `memory`, and `critical` messages go to `echo` (the console). [1] The memory buffer keeps the newest 1000 entries and is lost on reboot. [1] Debug messages are not logged by default. [1]

### Rules and actions

Logging is built from **rules** (`/system/logging`: which topics) and **actions** (`/system/logging/action`: where to send them). [1] A rule matches an entry only if the entry has all the listed topics; `!` excludes a topic. [1]

| Action target | Where entries go [1] |
| --- | --- |
| `memory` | RAM buffer (default 1000 lines) |
| `disk` | Files `log.0.txt`, `log.1.txt` in `/file` (1000 lines, 2 files by default) |
| `remote` | Syslog server; default `remote=0.0.0.0`, UDP 514 |
| `email`, `script` | Mail or a script |

### Debug and secrets

Debug messages can contain secrets; the manual warns, for example, that `ssh,debug` logs session keys. Keep debug output out of remote and email actions. [1]

### How it connects

```text
Q.01 logs ──► Q.02 resources and graphs ──► Q.03 SNMP ──► Q.05 packet capture when logs are not enough
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration + A.03 |
| Syslog server | Laptop `192.168.88.50` running a free syslog receiver (`TODO` name the tool used) |

## Method A — CLI

### Step 1 — Read and filter the log

```routeros
/log/print where topics~"interface|dhcp"
/log/print follow-only where topics~"critical"
```

[1] Unplug and replug a LAN cable while the second command runs (Ctrl+C to stop).

> [!CAUTION]
> **Privacy:** log lines contain MAC addresses, host names, and login addresses. Cover them. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-print.png)

### Step 2 — A separate debug buffer

```routeros
/system/logging/action/add name=dhcpdebug target=memory memory-lines=2000
/system/logging/add topics=dhcp,debug action=dhcpdebug
/log/print where buffer=dhcpdebug
```

[1] Renew the laptop's address (`ipconfig /renew`) and read the DHCP conversation.

`TODO` ![Step 2](images/cli/02-debug.png)

### Step 3 — Keep logs across reboots

```routeros
/system/logging/add topics=info action=disk
/file/print
```

[1] Reboot, then check that `log.0.txt` still holds entries from before the reboot.

`TODO` ![Step 3](images/cli/03-disk.png)

### Step 4 — Send warnings to a syslog server

```routeros
/system/logging/action/set remote remote=192.168.88.50 remote-log-format=syslog
/system/logging/add topics=warning action=remote
/log/warning "syslog test"
```

[1] The message appears in the laptop's syslog receiver.

`TODO` ![Step 4](images/cli/04-syslog.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Filtered print | Only interface/DHCP lines | `TODO` |
| Debug buffer | DORA messages for the laptop | `TODO` |
| After reboot | Old entries in `log.0.txt` | `TODO` |
| Syslog receiver | "syslog test" received | `TODO` |
