<div align="center">

# VLAN Concepts

*Chapter J — VLAN & Network Segmentation | Topic J.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

A VLAN splits one physical network into several separate layer-2 networks. This topic explains the 802.1Q tag, VLAN IDs, tagged and untagged ports, access and trunk ports, and the PVID, before J.02–J.04 configure them.

## Why This Matters

Offices separate staff, guests, cameras, and phones without buying a switch for each. VLANs are how that separation is done on shared cables, and how one router port can serve several networks.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [D.02 — Bridge Ports and Host Table](../../D-interface-and-bridge/02-bridge-ports-and-host-table/README.md) | Bridges and ports. |
| [F.04 — Virtual AP Guest Network](../../F-wireless-networking/04-virtual-ap-guest-network/README.md) | Separate subnets for separate groups. |

## Theory

### The 802.1Q tag

802.1Q inserts a 4-byte VLAN tag into the Ethernet header. [1] Each VLAN is its own network and normally its own subnet; traffic between VLANs needs a router. [1]

### VLAN IDs

The VLAN ID ranges from 1 to 4094. [1] IDs 0, 1, and 4095 should not be used in generic setups. [1]

### Tagged, untagged, access, trunk

| Term | Meaning |
| --- | --- |
| Tagged | The frame carries a VLAN tag on the wire. [2] |
| Untagged | The tag is removed before the frame leaves the port. [2] |
| Access port | Connects an end device; frames are untagged and belong to one VLAN. [2] |
| Trunk | A link between switches, or a switch and a router, carrying tagged frames for several VLANs. [1][2] |
| PVID | The VLAN that untagged frames arriving on a port are placed into. [2] |

### Filtering

With bridge VLAN filtering, the bridge checks which VLANs each port may send and receive. `ingress-filtering` drops frames for VLANs a port is not a member of, and `frame-types` limits whether a port accepts tagged, untagged, or both. [2] The manual recommends ingress filtering wherever possible. [2]

### MTU

The extra 4 bytes fit in the standard 1500-byte MTU on equipment that supports VLANs; the manual notes MTU 1496 as a workaround for some network cards. [1]

### How it connects

```text
J.01 concepts ──► J.02 VLAN interfaces on a router port ──► J.03 bridge VLAN filtering ──► J.04 inter-VLAN routing and isolation
```
