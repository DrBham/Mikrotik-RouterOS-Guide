<div align="center">

# Dynamic Routing and OSPF Concepts

*Chapter O — Dynamic Routing | Topic O.01*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

With static routes (G.01), every router must be told about every network by hand, and nothing adapts when a link fails. Dynamic routing protocols let routers tell each other which networks they know and recalculate paths automatically. This topic compares RIP, OSPF, and BGP and explains the OSPF ideas used in O.02 and O.03.

## Why This Matters

Beyond three or four routers, static routes become a maintenance problem and a source of mistakes. OSPF is the usual choice inside an organisation; BGP is how organisations and ISPs exchange routes.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [G.01 — Static Routes](../../G-routing-fundamentals/01-static-routes/README.md) | What dynamic routing replaces. |
| [B.07 — Gateway and Default Route](../../B-network-fundamentals/07-gateway-and-default-route/README.md) | Distance and longest match. |

## Theory

### Three protocols

| Protocol | Type | Notes |
| --- | --- | --- |
| RIP | Distance vector, counts hops | RouterOS implements RIPv2. [1] The maximum metric is 15; anything above is unreachable, so RIP cannot serve networks wider than 15 hops. [2] |
| OSPF | Link state, inside one AS | No hop limit; updates are sent only when the topology changes. [3] |
| BGP | Path vector, between ASes | Uses TCP port 179; `AS_PATH` prevents loops. [4] |

### OSPF terms

| Term | Meaning [3][5][6] |
| --- | --- |
| Router ID | Identifies the router; defaults to one of its IP addresses unless set. |
| Area | A logical group of routers; areas are identified by a 32-bit Area ID. |
| Backbone | Area `0.0.0.0`; every other area attaches to it. |
| Cost | Per-link value based on media speed; OSPF prefers the lowest total cost. |
| DR / BDR | Designated and backup designated router on broadcast networks; point-to-point links need none. |
| ABR / ASBR | Router between areas / router connected to an external network. |

### Neighbours

OSPF routers send Hello packets every 10 seconds by default; a neighbour that is silent for the dead interval (40 seconds by default) is considered down. [5] Two routers become neighbours only if they share the area, subnet, mask, and Hello and Dead intervals. [5] OSPF uses IP protocol 89 and the multicast groups `224.0.0.5` and `224.0.0.6`. [5] On a broadcast segment, the first router becomes DR and the second BDR; priority 0 means the router never takes part in the election. [5]

### Areas

Large networks are split into areas so that each router only holds detailed information for its own area. [6] The manual advises not to split areas with fewer than 50 routers. [6] Area types include standard, stub, totally stubby, and NSSA. [6]

### How it connects

```text
O.01 concepts ──► O.02 single-area OSPF ──► O.03 areas and default route ──► O.04 BGP between ASes
```

## How It Appears in RouterOS

| Protocol | Menus |
| --- | --- |
| OSPF | `/routing/ospf/instance`, `/routing/ospf/area`, `/routing/ospf/interface-template` [5] |
| RIP | `/routing/rip/instance`, `/routing/rip/interface-template` [2][7] |
| BGP | `/routing/bgp/instance`, `/routing/bgp/connection`, `/routing/bgp/session` [4] |

> [!WARNING]
> BGP is not supported on SMIPS devices such as the hAP lite. [4] Use CHR in GNS3 for this chapter.

```routeros
/routing/ospf/instance/print
/ip/route/print
```

`TODO` ![Routing menus](images/cli/01-routing-menus.png)

## Quick Recap

| Item | Meaning |
| --- | --- |
| RIP | Hops, max 15 [1][2] |
| OSPF | Link state, areas, cost [3] |
| BGP | Path vector, TCP 179, between ASes [4] |
| Area 0.0.0.0 | OSPF backbone [6] |
| Hello 10 s / dead 40 s | OSPF neighbour timers [5] |

## Key Takeaways

- Use OSPF inside your network, BGP between networks.
- OSPF neighbours must agree on area, subnet, and timers.

## References

1. [RouterOS Manual — RIP](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/unicast/rip/)
2. [RouterOS CLI Reference — routing/rip/instance](https://manual.mikrotik.com/docs/cli-reference/routing/rip/instance/)
3. [RouterOS Manual — Understanding OSPF](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/unicast/ospf/understanding-ospf/)
4. [RouterOS Manual — Understanding BGP](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/unicast/bgp/understanding-bgp/)
5. [RouterOS Manual — OSPF Neighbor Relationship](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/unicast/ospf/neighbour-relationship/)
6. [RouterOS Manual — OSPF Areas and Virtual Links](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/unicast/ospf/areas-and-virtual-links/)
7. [RouterOS CLI Reference — routing/rip/interface-template](https://manual.mikrotik.com/docs/cli-reference/routing/rip/interface-template/)

---

**Previous:** [N.04 — Netwatch](../../N-load-balancing-and-high-availability/04-netwatch/README.md) · **Next:** O.02 — Single-Area OSPF

[Back to the main learning journal](../../README.md)
