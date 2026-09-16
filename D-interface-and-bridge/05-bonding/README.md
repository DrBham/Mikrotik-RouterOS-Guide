<div align="center">

# Bonding

*Chapter D — Interface & Bridge | Topic D.05*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Bonding joins two or more Ethernet links into one virtual interface, for redundancy and, in some modes, more capacity. This topic connects two routers with two cables, bonds them, and pulls one cable out while traffic keeps flowing.

## Why This Matters

A single cable between a router and a core switch is a single point of failure. A bond keeps the link up when one cable, port, or transceiver fails.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [D.01 — Ethernet Interfaces and MTU](../01-ethernet-interfaces-and-mtu/README.md) | Link state of the member ports. |
| [D.03 — Spanning Tree Protocol](../03-spanning-tree-protocol/README.md) | Why two plain cables would form a loop. |

## Theory

### What a bond is

Bonding joins Ethernet-like interfaces into one virtual interface. [1] The member interfaces (slaves) must not have IP addresses of their own; the address goes on the bond. [1] The manual warns that in most modes a bond is not faster than one link for a single connection. [1]

### Modes

| Mode | Behaviour [1] |
| --- | --- |
| `balance-rr` | Default. The only mode that spreads one connection's packets over several links. |
| `active-backup` | One link active, the others wait; `primary=` chooses the active one. |
| `802.3ad` | LACP, negotiated with the switch. |
| `balance-xor` | Needs a static LAG on the switch. |
| `balance-tlb`, `balance-alb` | Balance outgoing traffic; `alb` also balances incoming IPv4 using ARP. |
| `broadcast` | Sends everything on all links. |
