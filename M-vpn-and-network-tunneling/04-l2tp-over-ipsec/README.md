<div align="center">

# L2TP over IPsec

*Chapter M — VPN & Network Tunneling | Topic M.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

L2TP is a PPP-based tunnel that Windows, macOS, Android, and iOS support without extra apps. On its own it does not encrypt, so it is combined with IPsec. This topic builds an L2TP/IPsec server with PPP accounts from L.01 and a MikroTik client.

## Why This Matters

When installing an app is not possible, L2TP/IPsec is often the VPN that works out of the box. It also reuses everything learned about PPP profiles and secrets.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [L.01 — PPPoE Server, Profiles, and Secrets](../../L-ppp-and-hotspot/01-pppoe-server-profiles-and-secrets/README.md) | PPP profiles and secrets. |
| [M.01 — VPN and Tunnel Concepts](../01-vpn-and-tunnel-concepts/README.md) | Encryption and MTU. |

## Theory

### Server and client

The manual's server: a pool, a PPP profile, a secret with `service=l2tp`, and the L2TP server enabled. [1]

```routeros
/ip/pool/add name=vpn-pool ranges=10.0.0.10-10.0.0.100
/ppp/profile/add name=vpn local-address=10.0.0.1 remote-address=vpn-pool
/ppp/secret/add name=alice password=Example-pass-1 profile=vpn service=l2tp
/interface/l2tp-server/server/set enabled=yes default-profile=vpn
```

A RouterOS client [1]:

```routeros
/interface/l2tp-client/add name=l2tp-office connect-to=203.0.113.1 user=alice password=Example-pass-1 disabled=no
```

### Adding IPsec

L2TP does not encrypt traffic, so the manual recommends IPsec on untrusted paths. [1] With `use-ipsec`, the server and client build the IPsec configuration themselves; both need the same `ipsec-secret`. [1] `use-ipsec=yes` accepts clients with or without IPsec; `required` refuses clients without it. [1]

```routeros
/interface/l2tp-server/server/set use-ipsec=required ipsec-secret=Example-ipsec-secret-1
/interface/l2tp-client/set l2tp-office use-ipsec=yes ipsec-secret=Example-ipsec-secret-1
```

[1]

### Firewall

The server's input chain must accept IKE (UDP 500 and 4500), ESP, and UDP 1701 when it comes through IPsec. [1]

### MTU

The default L2TP MTU is 1450. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Server | CHR or hAP lite, `TODO` version, H.02 firewall |
| Client | Second CHR (or Windows built-in VPN client) |
| Server address | Documentation address `203.0.113.1` in commands; replace with yours |
