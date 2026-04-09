<div align="center">

# Scheduler and Scripts

*Chapter E — DHCP & Network Services | Topic E.06*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

RouterOS can run commands on its own: at a time, at an interval, or at boot. This topic writes a small script that makes a nightly backup and export (A.07) and schedules it.

## Why This Matters

Backups that depend on remembering are backups that do not exist. A scheduled script also introduces RouterOS scripting, which later topics use for Netwatch and failover actions (Chapters N and Q).

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.07 — Backup, Export, and Restore](../../A-mikrotik-fundamentals/07-backup-export-and-restore/README.md) | What the script saves. |
| [E.04 — NTP, Clock, and Identity](../04-ntp-clock-and-identity/README.md) | The scheduler needs the right time. |

## Theory

### Scripts

A script is a list of RouterOS commands stored under `/system/script`. Commands are separated by `;` or a new line, braces `{}` hold the body, and `:local` and `:global` declare variables. [1] `:log info "text"` writes to the log. [1]

### The scheduler

The scheduler runs a named script, or an inline command in `on-event`, at a start time and interval, at startup (`start-time=startup`), or on selected weekdays. [2] An `interval` of `0s` runs the event once. [2] A reboot resets the run count. [2] Scheduled entries run with the permissions of the user who added them. [2]

The manual's example: a script that saves a backup and an export, scheduled daily at 03:00. [2]

```routeros
/system/script/add name=nightly-backup source={
    /system/backup/save name=nightly
    /export file=nightly
}
/system/scheduler/add name=nightly-backup start-time=03:00:00 interval=1d on-event=nightly-backup
```

> [!NOTE]
> For time-based firewall rules, the manual suggests the firewall `time` matcher instead of the scheduler. [2]

### How it connects

```text
E.06 scripts ──► Chapter N Netwatch up/down scripts ──► Chapter Q logging
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | E.04 done (NTP synchronised) |

> [!WARNING]
> The hAP lite has 16 MB of flash. Nightly files overwrite the same names, so they do not accumulate; do not add dates to the file names on this device.

## Method A — CLI

### Step 1 — Create the script

```routeros
/system/script/add name=nightly-backup source={
    /system/backup/save name=nightly
    /export file=nightly
    :log info "nightly backup done"
}
```

[1][2] The backup is saved without a password here; add `password=` for a real router (A.07).

`TODO` ![Step 1](images/cli/01-script.png)

### Step 2 — Run it once by hand

```routeros
/system/script/run nightly-backup
/file/print
/log/print where message~"nightly"
```

[2]

`TODO` ![Step 2](images/cli/02-run.png)

### Step 3 — Schedule it

```routeros
/system/scheduler/add name=nightly-backup start-time=03:00:00 interval=1d on-event=nightly-backup
/system/scheduler/print detail
```

[2]

`TODO` ![Step 3](images/cli/03-schedule.png)

### Step 4 — Test with a short interval

```routeros
/system/scheduler/set nightly-backup interval=2m start-time=startup
```

Wait two minutes and check `run-count` and the log, then set it back to `start-time=03:00:00 interval=1d`.

`TODO` ![Step 4](images/cli/04-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Files | `nightly.backup`, `nightly.rsc` | `TODO` |
| Log | "nightly backup done" | `TODO` |
| Scheduler run-count | Increases | `TODO` |
