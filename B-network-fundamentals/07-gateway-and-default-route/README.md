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

### Choosing a route

Two rules decide which route forwards a packet:

| Rule | Meaning |
| --- | --- |
| Longest prefix | If several routes match, the most specific one (largest netmask) is used. [2][3] |
| Distance | If several routes for the same destination are candidates, the one with the lowest distance becomes active. [2] |

Example from the manual: a static route with distance 1 wins over an OSPF route with distance 110 to the same prefix. [2]

### When there is no route

If a router has no route at all to a destination, not even a default route, it must drop the packet and send an ICMP Destination Unreachable (Network Unreachable) message back. [5] That is the `network unreachable` you see in ping and traceroute.

### TTL: protection against loops

Every router that forwards a datagram decreases its Time to Live by at least one, and a datagram whose TTL reaches zero is destroyed. [6] A routing loop therefore cannot keep a packet alive forever. Traceroute uses this on purpose: it sends probes with TTL 1, 2, 3, and so on, and each router that drops one reports back with an ICMP time exceeded message. [7]

### Checking the gateway

RouterOS can check whether a route's gateway still answers, with ARP probes, ICMP, or BFD, and stop using the route when it does not. [2] Chapter N uses this for failover between two internet connections.

### How it connects

```text
B.07 default route ──► C.02 from DHCP ──► C.03 static ──► C.04 from PPPoE
                    └► Chapter G static routing, Chapter N failover with check-gateway
```

## How It Appears in RouterOS

```routeros
/ip/route/print
/ip/route/print where dst-address=0.0.0.0/0
```

[2] With A.03 running you should see the connected LAN route, the connected wlan1 route, and a default route with the `d` (DHCP) flag and distance 1, added by the DHCP client. [8]

`TODO` ![Route table](images/cli/01-route-table.png)

```routeros
/tool/traceroute 8.8.8.8 count=1
```

[7] The first hop is the home router (your A.03 uplink gateway). The hAP lite does not show itself because the probe starts there.

> [!CAUTION]
> **Privacy:** traceroute shows your ISP's routers and possibly your public address. Cover the hops after your home router. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Traceroute](images/cli/02-traceroute.png)

On the laptop:

```powershell
ipconfig
route print -4
```

The laptop's default route points to `192.168.88.1`.

`TODO` ![Windows route](images/cli/03-windows-route.png)

## Quick Recap

| Item | Meaning |
| --- | --- |
| Gateway | Router on your own network that forwards to other networks [1] |
| `0.0.0.0/0` | Default route; used when nothing more specific matches [3][4] |
| Connected route | Automatic, one per IP network on an interface [2] |
| Longest prefix, then distance | How a route is chosen [2] |
| No route at all | Drop and ICMP network unreachable [5] |
| TTL | Decreased per hop; traceroute relies on it [6][7] |

## Key Takeaways

- A host only needs to know its own network and one gateway.
- The default route is the "everything else" route, chosen last.
- Network unreachable means a router had no route, not even a default one.

## References

1. [RFC 1122 — Requirements for Internet Hosts, §3.3.1](https://www.rfc-editor.org/rfc/rfc1122)
2. [RouterOS Manual — Routing decision](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/routing-decision/)
3. [RFC 1812 — Requirements for IP Version 4 Routers, §2.2.5.2 and §5.2.4.3](https://www.rfc-editor.org/rfc/rfc1812)
4. [RFC 4632 — Classless Inter-domain Routing (CIDR), §5.1](https://www.rfc-editor.org/rfc/rfc4632)
5. [RFC 1812 — §4.3.3.1 Destination Unreachable](https://www.rfc-editor.org/rfc/rfc1812)
6. [RFC 791 — Internet Protocol, §3.1 Time to Live](https://www.rfc-editor.org/rfc/rfc791)
7. [RouterOS Manual — Traceroute](https://manual.mikrotik.com/docs/diagnostics-monitoring-and-troubleshooting/traceroute/)
8. [RouterOS Manual — DHCP Client](https://manual.mikrotik.com/docs/network-management/dhcp/client/)

---

**Previous:** [B.06 — Public and Private IP Addresses](../06-public-and-private-ip-addresses/README.md) · **Next:** B.08 — DNS Concepts

[Back to the main learning journal](../../README.md)
