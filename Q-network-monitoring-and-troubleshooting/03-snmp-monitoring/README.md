<div align="center">

# SNMP Monitoring

*Chapter Q — Network Monitoring & Troubleshooting | Topic Q.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

SNMP lets a monitoring system read counters such as interface traffic, CPU, and uptime from many devices at once, and receive alerts (traps). This topic enables SNMP on the router, restricts it to one collector, and reads it from the laptop.

## Why This Matters

Tools like Zabbix, LibreNMS, PRTG, and The Dude collect data by SNMP. A router that answers SNMP only to the monitoring server is visible in those dashboards without being exposed to everyone.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [Q.02 — Resources, Profiler, and Graphing](../02-resources-profiler-and-graphing/README.md) | What is worth monitoring. |
| [H.02 — Protecting the Router](../../H-firewall-and-network-security/02-protecting-the-router/README.md) | Restricting management protocols. |

## Theory

### Agent, collector, communities

The router's SNMP agent answers on UDP 161; traps are sent to UDP 162. [1] SNMPv1/v2c use a community name as a simple password; SNMPv3 adds authentication and encryption. [1]

### The default community

The default `public` community is readable from any address (`::/0`), so restrict it or replace it. [1] Write access (`write-access=yes`) allows changing the identity, running scripts, and rebooting; the manual calls it dangerous. [1]

### The manual's configuration

```routeros
/snmp/set enabled=yes contact="NOC" location="Server room"
/snmp/community/add name=monit addresses=192.0.2.0/24 security=none read-access=yes
```

[1] Interface OIDs can be found with `/interface print oid where name=ether1`. [1]

### Traps

`trap-target`, `trap-community`, `trap-version`, and `trap-generators` configure traps; `/snmp/send-trap` sends a test trap. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version |
| Collector | Laptop `192.168.88.50` with an SNMP browser or `snmpwalk` (`TODO` tool) |

## Method A — CLI

### Step 1 — Enable SNMP with a restricted community

```routeros
/snmp/set enabled=yes contact="Lab" location="Desk"
/snmp/community/add name=monit addresses=192.168.88.50/32 security=none read-access=yes
/snmp/community/set [find name=public] addresses=192.168.88.50/32
```

[1]

`TODO` ![Step 1](images/cli/01-snmp.png)

### Step 2 — Find an OID

```routeros
/interface print oid where name=ether1
```

[1]

`TODO` ![Step 2](images/cli/02-oid.png)

### Step 3 — Read from the laptop

Query the router with community `monit` (for example the system name and ether1 counters).

`TODO` ![Step 3](images/cli/03-walk.png)

### Step 4 — Send a test trap

```routeros
/snmp/set trap-target=192.168.88.50 trap-community=monit trap-version=2
/snmp/send-trap oid=1.3.6.1.2.1.1.5.0 type=string value="test trap"
```

[1]

`TODO` ![Step 4](images/cli/04-trap.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Query from laptop | Values returned | `TODO` |
| Query from another address | No answer | `TODO` |
| Test trap | Received on UDP 162 | `TODO` |

## Reset Before Method B

```routeros
/snmp/community/remove [find name=monit]
/snmp/community/set [find name=public] addresses=::/0
/snmp/set enabled=no !trap-target
```

The manual's sample `/snmp/print` output shows `trap-target:` with no value. [1] A `!` before a parameter name in `set` unsets that parameter. [2]

> [!NOTE]
> The scripting manual's `!` example is for a menu with item IDs. `/snmp` has no IDs, so confirm in the lab that `trap-target` is empty again after this command.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — SNMP settings

**IP → SNMP**: Enabled, Contact, Location, Trap settings.

`TODO` ![Step 1](images/gui/01-snmp.png)

### Step 2 — Communities

**IP → SNMP → Communities**: add `monit` with Addresses `192.168.88.50/32`, Read Access.

`TODO` ![Step 2](images/gui/02-community.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| No answer from the router | Community address list excludes the collector, or SNMP disabled | Check `addresses`. [1] |
| Answers to anyone | `public` left open | Restrict or remove it. [1] |
| Traps not received | Collector firewall blocks UDP 162 | Allow it. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| UDP 161 / 162 | Queries / traps [1] |
| Community `addresses=` | Who may query [1] |
| `public` | Open by default; restrict it [1] |
| `write-access` | Dangerous [1] |

## Key Takeaways

- Enable SNMP read-only, for the collector's address only.
- Never leave the default community open.

## References

1. [RouterOS Manual — SNMP](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/snmp/)
2. [RouterOS Manual — Scripting (Common commands)](https://manual.mikrotik.com/docs/developer-guides/scripting/)

---

**Previous:** [Q.02 — Resources, Profiler, and Graphing](../02-resources-profiler-and-graphing/README.md) · **Next:** Q.04 — Traffic Flow

[Back to the main learning journal](../../README.md)
