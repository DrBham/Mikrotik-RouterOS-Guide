<div align="center">

# IP Cloud and DDNS

*Chapter E — DHCP & Network Services | Topic E.05*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Home and small-office connections often get a public address that changes. MikroTik's IP Cloud service gives every router a fixed DNS name that always points to its current public address. This topic enables it, reads the result, and explains what it exposes.

## Why This Matters

A VPN client (Chapter M) or a port forward (Chapter I) needs a stable name to connect to. IP Cloud provides one at no cost. It also reveals information about the router, so it should be enabled knowingly, not by accident.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.06 — Public and Private IP Addresses](../../B-network-fundamentals/06-public-and-private-ip-addresses/README.md) | Public addresses and CGN. |
| [B.08 — DNS Concepts](../../B-network-fundamentals/08-dns-concepts/README.md) | DNS names and TTL. |

## Theory

### How IP Cloud DDNS works

DDNS is off by default. When it is on, the name is the device's serial number in lower case followed by `.sn.mynetname.net`, with a 60-second TTL. [1] The router sends encrypted requests to MikroTik's cloud server on UDP port 15252 and reports the public address it is seen from. [1] Behind NAT, it updates every minute. [1]

With `ddns-enabled=auto`, DDNS stays on only while the Back To Home feature is enabled. [1]

### What it exposes

The name contains the device serial number. [1] The name itself opens no access, because the default firewall blocks connections from the WAN. [1] IP Cloud can also set the router's time and time zone. [1][2]

> [!CAUTION]
> **Privacy:** the DDNS name reveals the serial number and resolves to your public address. Cover both in every screenshot. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

### Behind CGN

If the ISP uses carrier-grade NAT (B.06), the public address the cloud sees belongs to the ISP, not to your router. The name then resolves, but inbound connections cannot reach you.

### How it connects

```text
E.05 DDNS name ──► Chapter M WireGuard endpoint ──► Chapter I port forwarding
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration + A.03 internet |
