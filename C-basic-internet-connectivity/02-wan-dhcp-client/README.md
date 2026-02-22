<div align="center">

# WAN via DHCP Client

*Chapter C — Basic Internet Connectivity | Topic C.02*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Starting from an empty configuration, this topic connects the router itself to the internet with a DHCP client on the WAN interface. At the end, the router can ping `8.8.8.8` and `google.com`. The LAN comes in C.05.

## Why This Matters

Most home and office uplinks hand out addresses by DHCP: a home router, a phone hotspot, most cable and fibre modems in router mode. This is the most common WAN type and the one the default configuration uses on ether1. [1]

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.01 — Lab Topology and Planning](../01-lab-topology-and-planning/README.md) | Interface plan and lists. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | DORA, leases, options. |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | Wireless station steps, if your WAN is wlan1. |

## Theory

### What the DHCP client does

The DHCP client on the WAN interface asks the upstream network for an address. When it gets a lease, RouterOS adds the address to the interface as a dynamic address and removes it when the lease is lost or the client is disabled. [2]

| Property | Default | Effect |
| --- | --- | --- |
| `add-default-route` | `yes` | Adds a default route through the received gateway (option 3), or the classless routes (option 121) if the server sends them. [2] |
| `default-route-distance` | `1` | Distance of that route (B.07). [2] |
| `use-peer-dns` | `yes` | Received DNS servers become the router's dynamic DNS servers. [2][3] |
| `use-peer-ntp` | | Received NTP servers go to the NTP client. [2] |

The status moves through `searching...` and `requesting...` to `bound`. [4]

### The router's own DNS

The router resolves names for itself using its DNS servers; static `servers` are tried before dynamic ones. [3] Serving DNS to the LAN (`allow-remote-requests`) is a separate setting used in C.05. [3]

### Marking the WAN

The interface that faces the internet goes into the `WAN` interface list, so C.06's NAT rule and Chapter H's firewall can refer to it by role. [5][6]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| WAN | `TODO` ether1 (cable) or wlan1 (station, see A.03) |
| Laptop | ether2, WinBox by MAC |
| Starting state | Empty configuration |
