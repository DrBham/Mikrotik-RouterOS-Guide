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

## How It Appears in RouterOS

### DHCP server (LAN)

A RouterOS DHCP server is three objects working together: an address pool, a network entry with the gateway and DNS to hand out, and the server bound to an interface. [3] The `setup` command creates all three at once. [3]

```routeros
/ip/pool/print
/ip/dhcp-server/network/print
/ip/dhcp-server/print
/ip/dhcp-server/lease/print
```

The pool must not include the router's own address. [3] Leases learned dynamically disappear when they expire or are released; static leases stay. [3]

> [!CAUTION]
> **Privacy:** the lease list shows MAC addresses and device host names. Cover them. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![DHCP server](images/cli/01-dhcp-server.png)

### DHCP client (WAN)

```routeros
/ip/dhcp-client/print detail
```

The status goes through `searching...` and `requesting...` to `bound`. [5] `add-default-route=yes` (the default) installs a default route from the router option, with distance 1, and `use-peer-dns=yes` passes the DNS servers to the router's resolver. [4][5]

`TODO` ![DHCP client](images/cli/02-dhcp-client.png)

On the laptop:

```powershell
ipconfig /renew
ipconfig
```

## Quick Recap

| Item | Meaning |
| --- | --- |
| DORA | Discover, Offer, Request, Ack [1] |
| UDP 67 / 68 | Server port / client port [1] |
| Options 1, 3, 6 | Mask, gateway, DNS [2] |
| Lease, T1, T2 | Address on loan; renew at 50%, rebind at 87.5% [1] |
| Pool + network + server | RouterOS DHCP server parts [3] |
| `bound` | RouterOS DHCP client has a lease [5] |

## Key Takeaways

- One DHCP exchange gives a client its address, gateway, and DNS together.
- A client stuck in `searching...` never got an offer: check the link and the server.
- Only one DHCP server should answer on a LAN.

## References

1. [RFC 2131 — Dynamic Host Configuration Protocol, §2, §3.1, §4.1, §4.4.1, §4.4.5](https://www.rfc-editor.org/rfc/rfc2131)
2. [RFC 2132 — DHCP Options and BOOTP Vendor Extensions, §3.3, §3.5, §3.8](https://www.rfc-editor.org/rfc/rfc2132)
3. [RouterOS Manual — DHCP Server](https://manual.mikrotik.com/docs/network-management/dhcp/server/)
4. [RouterOS Manual — DHCP Client](https://manual.mikrotik.com/docs/network-management/dhcp/client/)
5. [RouterOS CLI Reference — ip/dhcp-client](https://manual.mikrotik.com/docs/cli-reference/ip/dhcp-client/)

---

**Previous:** [B.08 — DNS Concepts](../08-dns-concepts/README.md) · **Next:** B.10 — NAT Concepts

[Back to the main learning journal](../../README.md)
