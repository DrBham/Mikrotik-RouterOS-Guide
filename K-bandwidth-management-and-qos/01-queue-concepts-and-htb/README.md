<div align="center">

# Queue Concepts and HTB

*Chapter K — Bandwidth Management & QoS | Topic K.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Queues decide how much bandwidth each user or kind of traffic gets, and what happens when the link is full. This topic explains limit-at and max-limit, shaping versus scheduling, the queue tree and simple queue menus, and HTB, the algorithm under all of them.

## Why This Matters

Without queues, one large download can make video calls unusable for everyone. With queues, the network still slows down under load, but fairly and predictably.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.01 — Firewall Concepts and Packet Flow](../../H-firewall-and-network-security/01-firewall-concepts-and-packet-flow/README.md) | Where queues sit in the packet flow; FastTrack. |

## Theory

### Two limits

| Term | Property | Meaning [1] |
| --- | --- | --- |
| CIR | `limit-at` | Committed rate: what a queue is guaranteed |
| MIR | `max-limit` | Maximum rate: what it may get when the parent has spare bandwidth |

### Shaping and scheduling

A shaper limits a flow to a rate and drops or delays the excess; a scheduler decides the order in which waiting packets leave. [1] RouterOS queues do both.

### Where queues run

An interface queue only handles traffic leaving that interface, so the WAN interface's queue handles uploads. [1] In the packet flow, queue trees and simple queues come at the end of postrouting. [2]

### HTB

RouterOS uses a modified Hierarchical Token Bucket (HTB) in simple queues and queue trees. [3] Each queue has a bucket of tokens; sending a packet spends tokens, and tokens spent by a child are also charged to its parent, so children borrow from parents. [3] Inner queues only distribute bandwidth; leaf queues hold the traffic. [3]

Rules of thumb from the manual: a parent's `limit-at` should be at least the sum of its children's, and its `max-limit` at least each child's `max-limit`. [3]

### Priority

Priority runs from 1 (highest) to 8 (lowest). It only applies to leaf queues that have a `max-limit`, and `limit-at` is given out regardless of priority. [3]

### FastTrack

Mangle rules do not see FastTracked packets, and the default configuration FastTracks established connections. [3] Simple queues do not limit FastTracked traffic, and queue trees with `parent=global` do not either; queue trees attached to an interface do. [4][5]

### How it connects

```text
K.01 HTB, CIR/MIR ──► K.02 simple queues ──► K.03 PCQ fair sharing ──► K.04 queue tree with mangle marks
```

## How It Appears in RouterOS

```routeros
/queue/interface/print
/queue/type/print
/queue/simple/print
/queue/tree/print
```

[1][6] `/queue/interface/print` shows the default queue of each interface.

`TODO` ![Queues](images/cli/01-queues.png)

## Quick Recap

| Item | Meaning |
| --- | --- |
| `limit-at` / `max-limit` | Guaranteed / maximum rate [1] |
| HTB | Parents lend tokens to children [3] |
| Priority 1–8 | Leaf queues with max-limit only [3] |
| FastTrack | Bypasses simple queues and `parent=global` trees [4][5] |

## Key Takeaways

- Guarantee with `limit-at`, cap with `max-limit`.
- Decide on FastTrack before building queues.

## References

1. [RouterOS Manual — Queues](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/)
2. [RouterOS Manual — Packet Flow in RouterOS](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/packet-flow-in-routeros/)
3. [RouterOS Manual — HTB (Hierarchical Token Bucket)](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/htb-hierarchical-token-bucket/)
4. [RouterOS Manual — Simple Queue](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/simple-queue/)
5. [RouterOS Manual — Queue Tree](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/queue-tree/)
6. [RouterOS Manual — Queue Types](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/queue-types/)

---

**Previous:** [J.04 — Inter-VLAN Routing and Isolation](../../J-vlan-and-network-segmentation/04-inter-vlan-routing-and-isolation/README.md) · **Next:** K.02 — Simple Queues

[Back to the main learning journal](../../README.md)
