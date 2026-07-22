<div align="center">

# IPsec Site-to-Site

*Chapter M — VPN & Network Tunneling | Topic M.06*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

IPsec is the standard VPN that firewalls and routers from most vendors support. This topic connects two offices with IKEv2 and a pre-shared key, following the manual's example, and handles the two classic traps: NAT and FastTrack.

## Why This Matters

When the other office uses a non-MikroTik firewall, IPsec is usually the common language. Its many parameters must match exactly on both sides, so a careful step-by-step build pays off.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [M.01 — VPN and Tunnel Concepts](../01-vpn-and-tunnel-concepts/README.md) | Keys and matching parameters. |
| [H.01 — Firewall Concepts and Packet Flow](../../H-firewall-and-network-security/01-firewall-concepts-and-packet-flow/README.md) | NAT, FastTrack, packet flow. |
| [E.04 — NTP, Clock, and Identity](../../E-dhcp-and-network-services/04-ntp-clock-and-identity/README.md) | IPsec needs synchronised clocks. |

## Theory

### The building blocks

| Menu | Holds [1] |
| --- | --- |
| `profile` | Phase 1 algorithms and timers |
| `proposal` | Phase 2 algorithms and lifetime |
| `peer` | Remote address, exchange mode, profile |
| `identity` | Authentication (pre-shared key, certificate) |
| `policy` | Which traffic is protected |

Phase 1 and phase 2 settings must match on both sides. [1] The manual also says to keep clocks synchronised with NTP. [1]

### The manual's Office 1 configuration

```routeros
/ip/ipsec/profile/add name=office2 hash-algorithm=sha256 enc-algorithm=aes-256 dh-group=ecp256
/ip/ipsec/proposal/add name=office2 auth-algorithms=sha256 enc-algorithms=aes-256-cbc pfs-group=ecp256
/ip/ipsec/peer/add name=office2 address=203.0.113.2/32 exchange-mode=ike2 profile=office2
/ip/ipsec/identity/add peer=office2 secret=Example-psk-1
/ip/ipsec/policy/add peer=office2 tunnel=yes src-address=10.1.202.0/24 dst-address=10.1.101.0/24 proposal=office2
```

[2] Office 2 is the mirror image. [2]

### NAT and FastTrack

IPsec policies match the original addresses, so traffic between the offices must not be masqueraded. The manual adds a `srcnat` `accept` rule above all other srcnat rules. [2] The default masquerade rule already excludes IPsec with `ipsec-policy=out,none`. [2] FastTrack bypasses IPsec policies, so tunnel traffic must be kept out of FastTrack. [2]

### Firewall

Accept UDP 500 and 4500 (IKE) and ESP from the peer before the input drop rule. [2]

## Lab Environment

```text
10.1.202.0/24 ─ [Office1] 203.0.113.1 ─── 203.0.113.2 [Office2] ─ 10.1.101.0/24
```

| Item | Value |
| --- | --- |
| Routers | Two CHRs in GNS3, `TODO` version, H.02/H.03 firewall |
| Addresses | As in the manual (documentation ranges) |

## Method A — CLI

### Step 1 — Phase 1 and 2, peer, identity

The first four commands on Office 1, and the mirror on Office 2. [2]

`TODO` ![Step 1](images/cli/01-ike.png)

### Step 2 — Policy

The policy line on both sides. [2]

`TODO` ![Step 2](images/cli/02-policy.png)

### Step 3 — NAT bypass

Office 1:

```routeros
/ip/firewall/nat/add chain=srcnat action=accept place-before=0 src-address=10.1.202.0/24 dst-address=10.1.101.0/24
```

Office 2 with the addresses swapped. [2]

`TODO` ![Step 3](images/cli/03-nat-bypass.png)

### Step 4 — Firewall

```routeros
/ip/firewall/filter add chain=input action=accept protocol=udp dst-port=500,4500 src-address=203.0.113.2 comment="accept IKE" place-before=0
/ip/firewall/filter add chain=input action=accept protocol=ipsec-esp src-address=203.0.113.2 comment="accept IPsec ESP" place-before=0
```

[2] Use the other office's address on each side.

`TODO` ![Step 4](images/cli/04-firewall.png)

### Step 5 — Check

```routeros
/ip/ipsec/active-peers/print
/ip/ipsec/installed-sa/print
/ping 10.1.101.1 src-address=10.1.202.1
```

[2] The peer shows `established`.

`TODO` ![Step 5](images/cli/05-check.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Active peer | `established` [2] | `TODO` |
| Installed SAs | Two (one per direction) | `TODO` |
| LAN-to-LAN ping | Works | `TODO` |
