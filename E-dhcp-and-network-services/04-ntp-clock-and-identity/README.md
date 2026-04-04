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
/system/ntp/client/set enabled=yes servers=0.pool.ntp.org,1.pool.ntp.org
/system/ntp/client/print
```

[4] Wait until `status: synchronized`.

`pool.ntp.org` is the NTP Pool project, a large virtual cluster of public time servers run by volunteers. [5] A lookup of `pool.ntp.org` usually returns servers in or close to your country, and the project's setup examples use the numbered names `0.pool.ntp.org` to `3.pool.ntp.org`. [6]

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

## Reset Before Method B

```routeros
/system/ntp/server/set enabled=no
/system/ntp/client/set enabled=no
/system/identity/set name=MikroTik
/system/clock/set time-zone-autodetect=yes
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Identity

**System → Identity**.

`TODO` ![Step 1](images/gui/01-identity.png)

### Step 2 — Clock

**System → Clock**: untick autodetect, choose the time zone.

`TODO` ![Step 2](images/gui/02-clock.png)

### Step 3 — NTP client and server

**System → NTP Client**: enable, add a server. **System → NTP Server**: enable.

`TODO` ![Step 3](images/gui/03-ntp.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| NTP stays `waiting` | No DNS or no internet, or UDP 123 blocked upstream | Ping the server name. |
| Time off by hours | Wrong time zone | Set the name manually. [2] |
| Log times wrong after reboot | NTP disabled, clock not kept | Enable the NTP client. [4] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/system/identity/set name=` | Router name [1] |
| `time-zone-name=` | Manual time zone [2] |
| `/system/ntp/client/set enabled=yes servers=` | Sync time [4] |
| `/system/ntp/server/set enabled=yes` | Serve time to the LAN [4] |

## Key Takeaways

- Name every router; it prevents mistakes in multi-router labs.
- Enable NTP before you rely on logs or certificates.

## References

1. [RouterOS Manual — Identity](https://manual.mikrotik.com/docs/system-information-and-utilities/identity/)
2. [RouterOS Manual — Clock](https://manual.mikrotik.com/docs/system-information-and-utilities/clock/)
3. [RouterOS Manual — Cloud](https://manual.mikrotik.com/docs/network-management/cloud/)
4. [RouterOS Manual — NTP](https://manual.mikrotik.com/docs/system-information-and-utilities/ntp/)
5. [NTP Pool Project — pool.ntp.org: the internet cluster of ntp servers](https://www.ntppool.org/en/)
6. [NTP Pool Project — How do I use pool.ntp.org?](https://www.ntppool.org/en/use.html)

---

**Previous:** [E.03 — DNS Static Entries and Forwarding](../03-dns-static-entries-and-forwarding/README.md) · **Next:** E.05 — IP Cloud and DDNS

[Back to the main learning journal](../../README.md)
