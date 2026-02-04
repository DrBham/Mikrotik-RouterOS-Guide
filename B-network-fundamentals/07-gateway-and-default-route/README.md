<div align="center">

# Gateway and Default Route

*Chapter B — Network Fundamentals & IP Addressing | Topic B.07*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

B.04 ended with a question: what does a host do when the destination is not on its own network? It sends the packet to a gateway. This topic explains gateways, routing tables, the default route `0.0.0.0/0`, and how a router picks one route out of several.

## Why This Matters

"Connected to Wi-Fi but no internet" is very often a missing or wrong default route or gateway. Every WAN setup in Chapter C ends with a default route, whether it comes from DHCP, a static entry, or PPPoE.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses and Subnet Masks](../04-ipv4-addresses-and-subnet-masks/README.md) | The local/remote decision. |
| [B.05 — Subnetting and CIDR](../05-subnetting-and-cidr/README.md) | Prefix length and longest match. |

## Theory

### Direct delivery or a gateway

If the destination is on a connected network, the datagram is sent directly to it; otherwise it has to be sent to a gateway on a connected network. [1] The gateway must itself be on the host's own network, because the host reaches it with ARP (B.03).

### The host's default gateway

When a host has no specific information about a destination, it chooses a "default" gateway and sends the datagram there. [1] A host generally needs at least one default gateway to get started. [1] On your laptop, this is the "Default Gateway" line in `ipconfig`, and on the A.03 LAN it is `192.168.88.1`.

### Routing tables

A router keeps a table of routes, each naming a destination prefix and where to send matching packets. RouterOS creates a connected route automatically for every IP network on an enabled interface. [2] Other routes are added by hand (static), learned from DHCP or PPP, or learned from routing protocols (Chapter O). [2]

### The default route

The default route is the route to all networks for which there is no explicit route; by definition its prefix length is zero. [3] In IPv4 it is written `0.0.0.0/0`. [4][2] Because every destination matches a `/0`, it is used only when nothing more specific matches.
