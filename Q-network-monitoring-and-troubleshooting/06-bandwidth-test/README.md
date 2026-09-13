<div align="center">

# Bandwidth Test

*Chapter Q — Network Monitoring & Troubleshooting | Topic Q.06*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Bandwidth Test measures throughput between two MikroTik devices, such as across a wireless link or a VPN tunnel. This topic runs a test between the hAP lite and a CHR with a dedicated test user, then closes the server again.

## Why This Matters

Speed tests on the internet measure the whole path. Bandwidth Test measures one link you control, which tells you whether a slow tunnel or wireless bridge is the bottleneck.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.06 — Users and Basic Router Security](../../A-mikrotik-fundamentals/06-users-and-basic-router-security/README.md) | A.06 disabled the bandwidth server. |
| [Q.02 — Resources, Profiler, and Graphing](../02-resources-profiler-and-graphing/README.md) | Watching CPU during the test. |

## Theory

### Client and server

One side runs the client (`/tool/bandwidth-test`); the other runs the server (`/tool/bandwidth-server`). [1] The server uses TCP 2000 for control and UDP 2000–65535 for data. [1] It requires authentication by default, and can be restricted with `allowed-addresses4` and `allowed-addresses6`. [1]

### A dedicated user

The manual creates a user that can only run tests: [1]

```routeros
/user/group/add name=btest policy=test,winbox
/user/add name=btest group=btest password=...
```

### Device mode

Device mode must allow `bandwidth-test`; the `home` and `basic` modes turn it off. [1]

### CPU limits

Generating traffic is CPU-heavy, so a small device may measure its own CPU limit instead of the link. Watch the CPU (Q.02) during the test.

## Lab Environment

| Item | Value |
| --- | --- |
| Server | CHR on the laptop's network, `TODO` version |
| Client | hAP lite, `TODO` version |
| Path | hAP lite LAN ↔ CHR (wired), later through the M.02 tunnel |
