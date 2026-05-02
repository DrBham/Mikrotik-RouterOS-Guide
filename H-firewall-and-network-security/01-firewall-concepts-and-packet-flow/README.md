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
