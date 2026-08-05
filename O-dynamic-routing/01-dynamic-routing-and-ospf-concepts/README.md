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
