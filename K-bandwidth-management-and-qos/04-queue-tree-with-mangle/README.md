<div align="center">

# Queue Tree with Mangle

*Chapter K — Bandwidth Management & QoS | Topic K.04*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Queue trees select traffic by packet mark instead of address, so they can prioritise by type of traffic. This topic marks voice traffic with mangle and gives it priority on the upload link, following the manual's example.

## Why This Matters

A video call breaks up when someone uploads a large file. Prioritising the call's packets on the upload link fixes that, and the same pattern works for games, VoIP, or remote desktop.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [K.01 — Queue Concepts and HTB](../01-queue-concepts-and-htb/README.md) | HTB and priority. |
| [H.01 — Firewall Concepts and Packet Flow](../../H-firewall-and-network-security/01-firewall-concepts-and-packet-flow/README.md) | Where mangle runs. |

## Theory

### Mangle marks

Mangle marks packets, connections, and routes; the marks exist only inside the router. [1] Marking the connection first and then its packets is cheaper than examining every packet. [1] `passthrough=no` stops processing further mangle rules for that packet. [1]

### Queue trees

A queue tree selects traffic by `packet-mark`; its `parent` is an interface, `global`, or another queue. [2] `no-mark` matches packets without a mark. [2] Queue trees have no order: each packet goes to the queue with its mark. [2]

### FastTrack and queue trees

FastTracked packets skip mangle and queues with `parent=global`; queues whose parent is an interface still limit them. [2] The manual restricts FastTrack to unmarked connections so marked traffic is queued: [2]

```routeros
/ip/firewall/filter set [find action=fasttrack-connection] connection-mark=no-mark
```

### The manual's voice example

```routeros
/ip/firewall/mangle add chain=forward dscp=46 connection-mark=no-mark action=mark-connection new-connection-mark=voice
/ip/firewall/mangle add chain=postrouting out-interface-list=WAN connection-mark=voice action=mark-packet new-packet-mark=voice passthrough=no
/queue/tree add name=upload parent=ether1 max-limit=20M
/queue/tree add name=upload-voice parent=upload packet-mark=voice priority=1 limit-at=2M max-limit=20M
/queue/tree add name=upload-other parent=upload packet-mark=no-mark priority=8 limit-at=18M max-limit=20M
```

[2] DSCP 46 is the value many VoIP phones put on voice packets; the manual's mangle page also shows a rule that sets it. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Router | MikroTik hAP lite or CHR, `TODO` version, WAN `ether1` |
| Upload link | Set `max-limit` a little below your real upload speed |
| Voice source | A softphone or a test that sets DSCP 46 (`TODO` tool) |

## Method A — CLI

### Step 1 — Exclude marked traffic from FastTrack

```routeros
/ip/firewall/filter set [find action=fasttrack-connection] connection-mark=no-mark
```

[2]

### Step 2 — Mangle rules

Enter the two mangle rules. [2] Use your WAN interface list.

`TODO` ![Step 2](images/cli/02-mangle.png)

### Step 3 — Queue tree

Enter the three queue tree rules with your upload speed. [2]

`TODO` ![Step 3](images/cli/03-tree.png)

### Step 4 — Test under load

Start a large upload from one device and a call from another.

```routeros
/queue/tree print stats
/ip/firewall/mangle/print stats
```

[2]

`TODO` ![Step 4](images/cli/04-stats.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Voice counters | Increase during the call | `TODO` |
| Call quality during upload | Clear | `TODO` |

## Reset Before Method B

```routeros
/queue/tree/remove [find]
/ip/firewall/mangle/remove [find new-connection-mark=voice]
/ip/firewall/mangle/remove [find new-packet-mark=voice]
/ip/firewall/filter set [find action=fasttrack-connection] !connection-mark
```

A `!` before a parameter name in `set` unsets that parameter, so the last line removes the `connection-mark` matcher from the FastTrack rule. [4]

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Mangle

**IP → Firewall → Mangle → +**: the two rules.

`TODO` ![Step 1](images/gui/01-mangle.png)

### Step 2 — Queue tree

**Queues → Queue Tree → +**: `upload` (parent ether1), `upload-voice` (parent upload, packet mark voice, priority 1), `upload-other` (packet mark no-mark, priority 8).

`TODO` ![Step 2](images/gui/02-tree.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Voice queue counts nothing | Traffic not marked DSCP 46, or FastTracked | Check mangle counters; Step 1. [2] |
| No effect on call quality | `max-limit` above the real link speed | Set it below; the queue must be the bottleneck. |
| Priority ignored | Queue without `max-limit`, or not a leaf | Priority needs both. [3] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `mark-connection` then `mark-packet` | Efficient marking [1] |
| `/queue/tree … packet-mark=` | Queue by mark [2] |
| `priority=1` + `limit-at` | Voice first, guaranteed share [2][3] |
| FastTrack `connection-mark=no-mark` | Keep marked traffic queued [2] |

## Key Takeaways

- Queue trees prioritise kinds of traffic; simple queues limit devices.
- The queue only helps if it is the slowest point on the link.

## References

1. [RouterOS Manual — Mangle](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/mangle/)
2. [RouterOS Manual — Queue Tree](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/queue-tree/)
3. [RouterOS Manual — HTB (Hierarchical Token Bucket)](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/htb-hierarchical-token-bucket/)
4. [RouterOS Manual — Scripting (Common commands)](https://manual.mikrotik.com/docs/developer-guides/scripting/)

---

**Previous:** [K.03 — PCQ Fair Sharing](../03-pcq-fair-sharing/README.md) · **Next:** Chapter L — PPP & Hotspot

[Back to the main learning journal](../../README.md)
