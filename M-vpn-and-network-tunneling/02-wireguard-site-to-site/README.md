<div align="center">

# WireGuard Site-to-Site

*Chapter M — VPN & Network Tunneling | Topic M.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic joins two office LANs with an encrypted WireGuard tunnel, following the manual's site-to-site example: an interface and key pair on each router, a peer pointing at the other side, tunnel addresses, routes, and a firewall rule.

## Why This Matters

WireGuard is simple, fast, and built into RouterOS 7. Two offices connected this way share printers, servers, and cameras as if they were one network, routed and encrypted.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [M.01 — VPN and Tunnel Concepts](../01-vpn-and-tunnel-concepts/README.md) | Keys and layer 3 tunnels. |
| [G.01 — Static Routes](../../G-routing-fundamentals/01-static-routes/README.md) | Routes to the remote LAN. |
| [H.02 — Protecting the Router](../../H-firewall-and-network-security/02-protecting-the-router/README.md) | Allowing the WireGuard port. |

## Theory

### Interface and keys

Adding a WireGuard interface generates its key pair automatically; only public keys are exchanged between routers. [1] `listen-port` sets the UDP port, 13231 in the manual. [1]

### Peers

A peer is identified by its public key and the local interface. [1] `allowed-address` decides which traffic may go through the tunnel to that peer; `endpoint-address` and `endpoint-port` say where to reach it. [1]

### Routing and firewall

The tunnel gets its own small subnet, and each router needs a route to the other LAN through the WireGuard interface. [1] The router's input firewall must accept the WireGuard UDP port from the peer. [1]

### The manual's Office 1 configuration

```routeros
/interface/wireguard add listen-port=13231 name=wireguard1
/interface/wireguard/peers add allowed-address=10.1.101.0/24,10.255.255.2/32 endpoint-address=192.168.80.1 endpoint-port=13231 interface=wireguard1 public-key="<Office2-public-key>"
/ip/address add address=10.255.255.1/30 interface=wireguard1
/ip/route add dst-address=10.1.101.0/24 gateway=wireguard1
/ip/firewall/filter add chain=input action=accept dst-port=13231 protocol=udp src-address=192.168.80.1 place-before=([find chain=input]->0)
```

[1] Office 2 mirrors it with `allowed-address=10.1.202.0/24,10.255.255.1/32`, endpoint `192.168.90.1`, address `10.255.255.2/30`, and a route to `10.1.202.0/24`. [1]

## Lab Environment

```text
LAN 10.1.202.0/24 ─ [Office1 CHR] WAN 192.168.90.1 ─── WAN 192.168.80.1 [Office2 CHR] ─ LAN 10.1.101.0/24
```

| Item | Value |
| --- | --- |
| Routers | Two CHRs in GNS3, `TODO` version, WANs on one shared segment |
| Addresses | As in the manual's example |

## Method A — CLI

### Step 1 — Interfaces and public keys

On both routers:

```routeros
/interface/wireguard add listen-port=13231 name=wireguard1
:put [/interface/wireguard/get [find name=wireguard1] public-key]
```

[1] Copy each router's public key.

> [!CAUTION]
> **Privacy:** never show private keys. Public keys are safe to share but are not needed in screenshots. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-keys.png)

### Step 2 — Peers

Office 1 and Office 2 peer commands from the theory section, with the other side's public key. [1]

`TODO` ![Step 2](images/cli/02-peers.png)

### Step 3 — Addresses, routes, firewall

The address, route, and firewall lines on both routers. [1]

`TODO` ![Step 3](images/cli/03-address-route-fw.png)

### Step 4 — Test

```routeros
/interface/wireguard/peers/print detail
/ping 10.1.101.1 src-address=10.1.202.1
```

The peer shows a recent handshake, and the ping crosses the tunnel.

`TODO` ![Step 4](images/cli/04-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Peer handshake | Recent | `TODO` |
| LAN-to-LAN ping | Works | `TODO` |
| Traceroute | One hop through the tunnel | `TODO` |
