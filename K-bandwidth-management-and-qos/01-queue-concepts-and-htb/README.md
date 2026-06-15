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
