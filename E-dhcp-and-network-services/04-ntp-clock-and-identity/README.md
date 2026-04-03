<div align="center">

# NTP, Clock, and Identity

*Chapter E — DHCP & Network Services | Topic E.04*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Two small settings make every later log, graph, and certificate readable: the router's name and its clock. This topic sets the identity, the time zone, and an NTP client, and turns the router into an NTP server for the LAN.

## Why This Matters

Logs with the wrong time are useless when you compare them with another device. IPsec (Chapter M) needs synchronised clocks, and certificates check dates. The identity appears in the CLI prompt and in WinBox Neighbors, which prevents configuring the wrong router.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | NTP needs to reach a time server. |

## Theory

### Identity

The default identity is `MikroTik`, up to 64 characters. It is shown in the prompt and in neighbour discovery, and used as the SNMP system name and the DHCP client host name. [1]

### The clock and time zone

Time zone autodetection is on by default and uses the location of the router's public address through MikroTik's cloud service. [2] At startup, the cloud can also set the time when NTP is disabled and `update-time` is enabled in `/ip/cloud`, but that time is only approximate. [2][3] A named time zone can be set manually; pressing Tab after `time-zone-name=` lists the names, which are case sensitive. [2]

### NTP

RouterOS includes an NTP client and server; both are disabled by default. [4] The client status goes from `stopped` to `waiting` to `synchronized`. [4] Servers can be addresses or names, and servers learned from DHCP (`use-peer-ntp`) appear as dynamic entries. [4] The NTP server can serve the router's own time to the LAN. [4]

### How it connects

```text
E.04 correct time ──► Chapter Q logs and graphs ──► Chapter M IPsec ──► certificates
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration + A.03 internet |
| Time zone | `Asia/Jakarta` (change to yours) |

## Method A — CLI

### Step 1 — Set the identity

```routeros
/system/identity/set name=LAB-hAP-lite
/system/identity/print
```

[1] The prompt changes to the new name.

`TODO` ![Step 1](images/cli/01-identity.png)

### Step 2 — Check and set the clock

```routeros
/system/clock/print
/system/clock/set time-zone-autodetect=no time-zone-name=Asia/Jakarta
```

[2] Use Tab after `time-zone-name=` to find yours.

`TODO` ![Step 2](images/cli/02-clock.png)

### Step 3 — Enable the NTP client

```routeros
/system/ntp/client/set enabled=yes servers=pool.ntp.org
/system/ntp/client/print
```

[4] Wait until `status: synchronized`.

> [!WARNING]
> **Source needed:** `pool.ntp.org` is a public NTP pool, not named on the RouterOS NTP page. Its use as an example comes from [ntppool.org](https://www.ntppool.org/); confirm before publishing.

`TODO` ![Step 3](images/cli/03-ntp-client.png)

### Step 4 — Serve time to the LAN

```routeros
/system/ntp/server/set enabled=yes
```

[4] On Windows, `w32tm /stripchart /computer:192.168.88.1 /samples:3` queries it.

`TODO` ![Step 4](images/cli/04-ntp-server.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Prompt | `[admin@LAB-hAP-lite]` [1] | `TODO` |
| NTP client | `synchronized` [4] | `TODO` |
| `/system/clock/print` | Correct local time | `TODO` |
