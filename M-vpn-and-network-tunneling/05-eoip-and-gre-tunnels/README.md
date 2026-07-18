<div align="center">

# EoIP and GRE Tunnels

*Chapter M — VPN & Network Tunneling | Topic M.05*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

EoIP and GRE are lightweight tunnels without built-in encryption. GRE carries IP packets between two routed networks; EoIP carries whole Ethernet frames so two sites can share one bridged LAN. This topic builds a GRE tunnel first, then an EoIP tunnel bridged into the LAN.

## Why This Matters

GRE is the standard tunnel between vendors and is often combined with IPsec or routing protocols. EoIP is how MikroTik networks stretch one LAN across a link, for example to extend a network over a point-to-point wireless bridge.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [M.01 — VPN and Tunnel Concepts](../01-vpn-and-tunnel-concepts/README.md) | Layer 2 vs layer 3, MTU. |
| [D.02 — Bridge Ports and Host Table](../../D-interface-and-bridge/02-bridge-ports-and-host-table/README.md) | Bridging the EoIP interface. |

## Theory

### GRE

GRE is a layer-3 tunnel that carries only IPv4 and IPv6 packets. [1] It is stateless, with a keepalive of 10 seconds and 10 retries by default. [1] It adds 24 bytes over IPv4, so the tunnel MTU on a 1500-byte link is 1476. [1] The manual's two-site example: [1]

```routeros
/interface/gre/add name=myGre remote-address=192.168.90.1 local-address=192.168.80.1
/ip/address/add address=172.16.1.1/30 interface=myGre
/ip/route/add dst-address=10.1.202.0/24 gateway=172.16.1.2
```

### EoIP

EoIP is a MikroTik protocol that carries full Ethernet frames, including broadcasts and non-IP traffic, over IP using GRE encapsulation (IP protocol 47). [2] Both ends must use the same `tunnel-id`. [2] It adds 42 bytes, giving an MTU of 1458 on a 1500-byte path. [2] Each EoIP interface on a bridge needs its own MAC address. [2]

```routeros
/interface/eoip/add name=eoip-remote remote-address=10.0.0.2 tunnel-id=1
/interface/bridge/port/add bridge=bridge1 interface=eoip-remote
```

[2]

### Encryption

Both tunnels can be encrypted with `ipsec-secret` on both ends; for EoIP, `allow-fast-path=no` must be set first. [2]

## Lab Environment

```text
LAN-A ─ [R1] 192.168.80.1 ─── 192.168.90.1 [R2] ─ LAN-B
```

| Item | Value |
| --- | --- |
| Routers | Two CHRs in GNS3, `TODO` version |
| Part 1 (GRE) | LAN-A `10.1.101.0/24`, LAN-B `10.1.202.0/24` |
| Part 2 (EoIP) | Both LANs in one subnet `10.1.1.0/24` via bridge1 |
