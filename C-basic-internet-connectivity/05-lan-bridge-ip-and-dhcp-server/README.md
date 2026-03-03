<div align="center">

# LAN Bridge, IP, and DHCP Server

*Chapter C — Basic Internet Connectivity | Topic C.05*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The router is online (C.02–C.04). Now the LAN side: join ether2–ether4 in a bridge, give the bridge the LAN address, run a DHCP server so clients configure themselves, and let the router answer their DNS queries.

## Why This Matters

This is the half of the default configuration that every client depends on. Without it, a laptop plugged into the router gets `169.254.x.x` (B.06) and cannot even reach the router by IP.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.02 — WAN via DHCP Client](../02-wan-dhcp-client/README.md) | A working WAN and the `LAN` list. |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | Bridges. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | Pool, options, leases. |

## Theory

### The bridge

To combine several ports into one network, RouterOS creates a bridge interface and adds the ports to it. [1] The bridge learns which MAC address is behind which port (B.03). [1] The LAN address goes on the bridge, as the manual's DHCP example does with `interface=bridge1`. [2]

### The DHCP server's three parts

| Part | Holds | Command menu [2] |
| --- | --- | --- |
| Pool | The range of addresses to lease | `/ip/pool` |
| Network | Gateway and DNS server sent to clients (options 3 and 6) | `/ip/dhcp-server/network` |
| Server | Which interface it answers on, and which pool it uses | `/ip/dhcp-server` |

The `setup` command builds all three at once, but needs the IP address on the interface first. [2] The pool must not contain the router's own address. [2] On a bridge, the server needs at least one real port that receives packets. [2]

### DNS for the LAN

Clients will be told to use `192.168.88.1` as their DNS server. For the router to answer them, `allow-remote-requests=yes` must be set; it makes the router answer on port 53 on all its addresses. [3] That is why C.06/Chapter H must block DNS from the WAN. [3]

### Marking the LAN

The bridge joins the `LAN` interface list. Adding a bridge to a list is not the same as adding its ports. [4]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | C.02 finished (WAN online, `WAN`/`LAN` lists) |
| Laptop | ether2, WinBox by MAC, Ethernet set to automatic (DHCP) |

> [!WARNING]
> Adding ether2 to the bridge can interrupt the MAC WinBox session for a moment. Reconnect from Neighbors if it drops.
