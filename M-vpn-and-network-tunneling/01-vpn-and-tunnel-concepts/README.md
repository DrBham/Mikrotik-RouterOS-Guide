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
