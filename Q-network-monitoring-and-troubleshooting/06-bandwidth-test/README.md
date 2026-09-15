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

## Method A — CLI

### Step 1 — Server side (CHR)

```routeros
/user/group/add name=btest policy=test,winbox
/user/add name=btest group=btest password="YOUR_TEST_PASSWORD"
/tool/bandwidth-server/set enabled=yes allowed-addresses4=192.168.88.0/24
```

[1]

`TODO` ![Step 1](images/cli/01-server.png)

### Step 2 — Run a test (hAP lite)

```routeros
/tool/bandwidth-test address=<CHR-address> user=btest password="YOUR_TEST_PASSWORD" protocol=tcp direction=both duration=10s
```

[1]

`TODO` ![Step 2](images/cli/02-test.png)

### Step 3 — Compare UDP and watch CPU

Repeat with `protocol=udp local-tx-speed=50M remote-tx-speed=50M` while `/system/resource/monitor` runs in a second session. [1][2]

`TODO` ![Step 3](images/cli/03-udp-cpu.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| TCP both directions | `TODO` Mbps | `TODO` |
| UDP 50M | `TODO` Mbps, loss `TODO` | `TODO` |
| hAP lite CPU | `TODO` % | `TODO` |

## Reset Before Method B

On the CHR:

```routeros
/tool/bandwidth-server/set enabled=no
/user/remove btest
/user/group/remove btest
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Server

On the CHR: **Tools → BTest Server**: Enabled, Authenticate; create the `btest` group and user in **System → Users**.

`TODO` ![Step 1](images/gui/01-server.png)

### Step 2 — Client

On the hAP lite: **Tools → Bandwidth Test**: Test To, Protocol, Direction, User, Password, **Start**.

`TODO` ![Step 2](images/gui/02-client.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Connection refused | Server disabled (A.06) or address not allowed | Step 1. [1] |
| Not available | Device mode `home` or `basic` [1] | Check device mode. |
| Result far below the link speed | CPU at 100% | Test between faster devices. [2] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/tool/bandwidth-test address= protocol= direction=` | Client [1] |
| `/tool/bandwidth-server` + `allowed-addresses4` | Server, restricted [1] |
| TCP 2000 + UDP 2000–65535 | Ports [1] |
| `btest` user with `test` policy | Least privilege [1] |

## Key Takeaways

- Measure one link at a time with Bandwidth Test.
- Turn the server off again after testing.

## References

1. [RouterOS Manual — Bandwidth Test](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/bandwidth-test/)
2. [RouterOS Manual — Resource](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/resource/)

---

**Previous:** [Q.05 — Packet Sniffer and Wireshark](../05-packet-sniffer-and-wireshark/README.md) · **Next:** End of the roadmap. Return to the [main learning journal](../../README.md).

[Back to the main learning journal](../../README.md)
