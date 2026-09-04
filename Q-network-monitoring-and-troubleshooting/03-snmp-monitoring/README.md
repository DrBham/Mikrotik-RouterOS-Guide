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
