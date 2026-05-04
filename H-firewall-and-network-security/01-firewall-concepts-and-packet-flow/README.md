<div align="center">

# Firewall Concepts and Packet Flow

*Chapter H — Firewall & Network Security | Topic H.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Before writing firewall rules, you need to know where a packet passes through the router and in what order RouterOS looks at it. This topic explains the filter chains, rule order, connection states, and the packet flow that ties firewall, NAT, mangle, and queues together.

## Why This Matters

A rule in the wrong chain never matches, and a rule in the wrong position never runs. Most firewall mistakes are not wrong rules but rules in the wrong place, and the packet flow diagram explains why.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.10 — NAT Concepts](../../B-network-fundamentals/10-nat-concepts/README.md) | NAT and connection tracking. |
| [C.06 — NAT Masquerade](../../C-basic-internet-connectivity/06-nat-masquerade/README.md) | First filter rules. |

## Theory

### Three filter chains

| Chain | Sees [1] |
| --- | --- |
| `input` | Packets addressed to the router itself |
| `forward` | Packets passing through the router |
| `output` | Packets the router itself sends |

A packet passing through the router never appears in `input` or `output`. [1] Protecting the router is an `input` job; protecting the LAN is a `forward` job.

### Rule order

Rules are checked from top to bottom, and the first matching rule with `accept`, `drop`, or `reject` decides. [1] A packet that matches no rule is accepted. [1] The manual describes two styles: accept what you need and drop the rest (more secure), or drop only what you do not want. [1]

### Connection states

Connection tracking groups packets into connections and gives each packet a state [2]:

| State | Meaning [2] |
| --- | --- |
| `new` | Starts a connection, or only one direction seen so far |
| `established` | Belongs to a connection seen in both directions |
| `related` | Not in the list, but related to one (e.g. an ICMP error, FTP data) |
| `invalid` | Belongs to no connection and cannot start one |
| `untracked` | Excluded from tracking by a RAW rule |

The default configuration starts both `input` and `forward` with "accept established, related, untracked" and "drop invalid", and the manual says to keep them first. [2] Most packets are then accepted by the first rule, and only `new` connections reach the rules below.

### FastTrack

A `fasttrack-connection` rule sends established TCP and UDP connections through a fast path that skips the firewall, connection tracking, and queues. [2] It speeds up a router but means those packets are not seen by later filter, mangle, or queue rules. [2]

### Packet flow

For a packet passing through the router, RouterOS applies these stages in order [3]:

```text
Prerouting:  RAW prerouting → connection tracking → mangle prerouting → NAT dstnat
Routing decision
Forward:     TTL decrement → mangle forward → filter forward
Postrouting: mangle postrouting → NAT srcnat → queue tree → simple queues
```

For a packet to the router, routing is followed by `mangle input` and `filter input`. [3] Destination NAT happens before the routing decision and before the filter, so forward rules see the translated (inside) address. [3]

### How it connects

```text
H.01 chains + order ──► H.02 input rules ──► H.03 forward rules ──► H.04 RAW and address lists
                    └─► Chapter I dstnat runs before filter; Chapter K queues run after
```

## How It Appears in RouterOS

```routeros
/ip/firewall/filter/print
/ip/firewall/filter/print stats
/ip/firewall/connection/print count-only
```

[1][2] On a default configuration, identify the established/related rule, the invalid drop, the FastTrack rule, and the "drop all not coming from LAN" rule.

`TODO` ![Default firewall](images/cli/01-default-firewall.png)

## Quick Recap

| Item | Meaning |
| --- | --- |
| `input`, `forward`, `output` | To, through, from the router [1] |
| First match decides; no match accepts | Rule order [1] |
| new / established / related / invalid / untracked | Connection states [2] |
| FastTrack | Skips firewall and queues for established flows [2] |
| dstnat before filter, srcnat after | Packet flow [3] |

## Key Takeaways

- Put "accept established, related" and "drop invalid" first.
- Protect the router in `input`, the LAN in `forward`.
- End with a drop rule, or everything not matched is accepted.

## References

1. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)
2. [RouterOS Manual — Connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/)
3. [RouterOS Manual — Packet Flow in RouterOS](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/packet-flow-in-routeros/)

---

**Previous:** [G.02 — Policy Routing](../../G-routing-fundamentals/02-policy-routing/README.md) · **Next:** H.02 — Protecting the Router

[Back to the main learning journal](../../README.md)
