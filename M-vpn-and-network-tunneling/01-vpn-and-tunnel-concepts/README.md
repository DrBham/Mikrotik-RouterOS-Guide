<div align="center">

# VPN and Tunnel Concepts

*Chapter M — VPN & Network Tunneling | Topic M.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

A tunnel carries packets of one network inside packets of another, so two distant networks behave as if connected directly. A VPN is a tunnel that is also authenticated and, usually, encrypted. This topic compares the tunnel types RouterOS offers and when each fits.

## Why This Matters

Branch offices, working from home, and managing routers safely all depend on tunnels. Choosing the wrong one gives problems with speed, MTU, security, or NAT that are hard to fix later.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.02 — OSI and TCP/IP Models](../../B-network-fundamentals/02-osi-and-tcp-ip-models/README.md) | Layer 2 vs layer 3, encapsulation. |
| [G.01 — Static Routes](../../G-routing-fundamentals/01-static-routes/README.md) | Routing traffic into a tunnel. |

## Theory

### Layer 2 and layer 3 tunnels

| Type | Carries | RouterOS examples |
| --- | --- | --- |
| Layer 3 | IP packets only; each side is a separate subnet, joined by routes | GRE carries IPv4 and IPv6 only. [1] WireGuard. [2] |
| Layer 2 | Whole Ethernet frames, including broadcasts; both sides can be one bridged LAN | EoIP carries full Ethernet frames over IP. [3] |

### Encryption

GRE and EoIP do not encrypt; both can be protected with an `ipsec-secret`. [3][4] L2TP does not encrypt either, and the manual says to use IPsec on untrusted paths. [5] WireGuard and IPsec encrypt by design. [2][4]

### Overhead and MTU

Every tunnel adds headers, which lowers the usable MTU:

| Tunnel | Overhead | MTU on a 1500-byte path |
| --- | --- | --- |
| GRE over IPv4 | 24 bytes | 1476 [1] |
| EoIP | 42 bytes | 1458 [3] |
| L2TP | | 1450 default [5] |

Fragmentation or MSS clamping (C.04) handles the difference.

### Keys and peers

WireGuard identifies each peer by its public key; the private key never leaves the router, and only public keys are exchanged. [2] IPsec uses a pre-shared key or certificates, with phase 1 and phase 2 parameters that must match on both sides. [4]

### Which to choose

| Need | Good fit |
| --- | --- |
| Join two offices, both MikroTik | WireGuard (M.02) |
| Phones and laptops connecting in | WireGuard (M.03) or L2TP/IPsec (M.04) |
| One LAN across two sites | EoIP, encrypted (M.05) |
| Third-party firewall at the other end | IPsec IKEv2 (M.06) |

### How it connects

```text
M.01 concepts ──► M.02–M.06 builds ──► E.05 DDNS name as the endpoint ──► H.02 allow the VPN port in input
```

## How It Appears in RouterOS

| Tunnel | Menu |
| --- | --- |
| WireGuard | `/interface/wireguard`, `/interface/wireguard/peers` [2] |
| L2TP | `/interface/l2tp-server`, `/interface/l2tp-client` [5] |
| EoIP / GRE | `/interface/eoip`, `/interface/gre` [3][1] |
| IPsec | `/ip/ipsec/...` [4] |

```routeros
/interface/print where type~"wg|gre|eoip|l2tp"
```

`TODO` ![Tunnel menus](images/cli/01-tunnels.png)

## Quick Recap

| Item | Meaning |
| --- | --- |
| L3 tunnel | Routed; IP only [1] |
| L2 tunnel | Bridged; full Ethernet [3] |
| GRE/EoIP/L2TP | No encryption without IPsec [3][4][5] |
| Overhead | Lower MTU: GRE 1476, EoIP 1458 [1][3] |

## Key Takeaways

- Decide layer 2 or layer 3 first, then whether you need encryption.
- Every tunnel lowers the MTU; plan for it.

## References

1. [RouterOS Manual — GRE](https://manual.mikrotik.com/docs/virtual-private-networks/gre/)
2. [RouterOS Manual — WireGuard](https://manual.mikrotik.com/docs/virtual-private-networks/wireguard/)
3. [RouterOS Manual — EoIP](https://manual.mikrotik.com/docs/virtual-private-networks/eoip/)
4. [RouterOS Manual — IPsec](https://manual.mikrotik.com/docs/virtual-private-networks/ipsec/)
5. [RouterOS Manual — L2TP](https://manual.mikrotik.com/docs/virtual-private-networks/l2tp/)

---

**Previous:** [L.03 — HotSpot Users and Profiles](../../L-ppp-and-hotspot/03-hotspot-users-and-profiles/README.md) · **Next:** M.02 — WireGuard Site-to-Site

[Back to the main learning journal](../../README.md)
