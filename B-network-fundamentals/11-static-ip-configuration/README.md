<div align="center">

# Static IP Configuration

*Chapter B — Network Fundamentals & IP Addressing | Topic B.11*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic puts Chapter B into practice. We take ether3 out of the LAN bridge, give it its own static subnet, give the laptop a static address on that subnet, and test which traffic works and why.

## Why This Matters

Routers, servers, printers, and point-to-point links use static addresses so they never change. Doing it by hand once makes every field in B.04 and B.07 concrete: the address, the prefix, the network, and the gateway the client must point to.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses and Subnet Masks](../04-ipv4-addresses-and-subnet-masks/README.md) | Address, mask, network. |
| [B.07 — Gateway and Default Route](../07-gateway-and-default-route/README.md) | Why the client needs a gateway. |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | Internet access for the last test. |

## Theory

### Addresses in RouterOS

An address is entered as address plus prefix, for example `192.168.88.1/24`, and RouterOS calculates the `network`. [1] Adding an address also creates a connected route to its network. [2] The default configuration puts the LAN address on the bridge, not on the individual ports. [3]

### The plan

| Item | Value | Why |
| --- | --- | --- |
| Router ether3 | `10.10.10.1/24` | A new subnet, outside the LAN `192.168.88.0/24` (B.05) |
| Laptop | `10.10.10.2`, mask `255.255.255.0` | Same network as ether3 (B.04) |
| Laptop gateway | `10.10.10.1` | The router address on this subnet (B.07) |
| Laptop DNS | `10.10.10.1` | The router's resolver (B.08) |

ether3 must leave the LAN bridge first: a port that is a bridge member belongs to the bridge's network. [3]

### Will it reach the internet?

The router's NAT rule from the default configuration translates traffic going out of the WAN interface list. [4] It does not care which LAN subnet the packet came from unless the rule limits `src-address`, so traffic from `10.10.10.0/24` should also be translated. Check your rule in Step 1.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration + A.03 (internet over wlan1) |
| Laptop | Moves from ether2 to **ether3** during the lab |

> [!WARNING]
> Keep the laptop on ether2 until Step 3 is done, so you do not lose access to the router.
