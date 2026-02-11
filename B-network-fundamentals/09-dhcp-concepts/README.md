<div align="center">

# DHCP Concepts

*Chapter B — Network Fundamentals & IP Addressing | Topic B.09*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Every device on a network needs an address, a mask, a gateway, and DNS servers. The Dynamic Host Configuration Protocol (DHCP) hands all of them out automatically. This topic explains the four-message exchange, leases, the options that carry the gateway and DNS, and both sides of DHCP on a MikroTik: the server on the LAN and the client on the WAN.

## Why This Matters

Your laptop got `192.168.88.x` from the hAP lite's DHCP server in A.02, and the hAP lite got its wlan1 address from the home router's DHCP server in A.03. Knowing how that exchange works explains the `searching...` and `bound` states, the `169.254.x.x` fallback from B.06, and why a second DHCP server on the same LAN causes chaos.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses and Subnet Masks](../04-ipv4-addresses-and-subnet-masks/README.md) | Broadcast and "this host" addresses. |
| [B.07 — Gateway and Default Route](../07-gateway-and-default-route/README.md) | The gateway a client receives. |
| [B.08 — DNS Concepts](../08-dns-concepts/README.md) | The DNS servers a client receives. |

## Theory

### What DHCP gives a client

DHCP delivers a network address and the other configuration parameters a host needs to operate. [1] The parameters travel as options; three of them matter for every LAN [2]:

| Option | Code [2] | Carries |
| --- | --- | --- |
| Subnet Mask | 1 | The client's subnet mask |
| Router | 3 | Routers (gateways) on the client's subnet, in order of preference |
| Domain Name Server | 6 | DNS servers, in order of preference |

### The four messages (DORA)

| Step | Message [1] | Sent by |
| --- | --- | --- |
| 1 | DHCPDISCOVER | Client, broadcast on its local subnet |
| 2 | DHCPOFFER | Each server that can offer an address |
| 3 | DHCPREQUEST | Client, asking for the chosen offer |
| 4 | DHCPACK | Server, confirming the address and parameters |

DHCP uses UDP. Clients send to the DHCP server port 67, and servers reply to the client port 68. [1] Before it has an address, a client sends with source address `0` and broadcasts the DISCOVER to `255.255.255.255`. [1] This is the "this host on this network" and limited broadcast address from B.04.

Other messages include DHCPNAK (the server refuses), DHCPDECLINE (the address is already in use), DHCPRELEASE, and DHCPINFORM. [1]

### Leases

An address is given for a limited time, a lease. The client can extend it or give it back early. [1] By default the client tries to renew at T1, half the lease time, and asks any server (rebinding) at T2, 87.5% of the lease time. [1]

### How it connects

```text
LAN side:  hAP lite DHCP server ──► laptop gets 192.168.88.x, gateway .1, DNS .1   (C.05)
WAN side:  home router DHCP server ──► hAP lite wlan1 gets an address and a default route  (A.03, C.02)
```
