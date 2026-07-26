<div align="center">

# WAN Failover

*Chapter N — Load Balancing & High Availability | Topic N.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

With two internet connections, the second should take over when the first fails. This topic builds failover with route distances and `check-gateway`, then improves it by checking hosts on the internet rather than only the ISP's gateway, as the manual describes.

## Why This Matters

A second line is only useful if traffic moves to it automatically. Checking only the gateway misses the most common failure: the ISP's router answers, but the internet behind it does not.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [G.01 — Static Routes](../../G-routing-fundamentals/01-static-routes/README.md) | Distance and check-gateway. |
| [C.06 — NAT Masquerade](../../C-basic-internet-connectivity/06-nat-masquerade/README.md) | NAT on each WAN. |

## Theory

### Method 1: distance and check-gateway

The route with the lower distance is used while it is active. With `check-gateway=ping`, the router pings the gateway; if it stops answering, the route becomes inactive and the next distance takes over, in about 20 seconds with default timing. [1]

```routeros
/ip/route add distance=1 gateway=10.111.0.1 check-gateway=ping
/ip/route add distance=2 gateway=10.112.0.1 check-gateway=ping
```

[1]

### The limitation

A gateway that answers does not prove the provider has internet access. [1]

### Method 2: check a host behind each link

The manual's second method pins one public host to each link with a host route of `scope=10`, then uses that host as a recursive gateway for the default route with `target-scope=11`. If the host stops answering through that link, the default route through it fails over. [1]

```routeros
/ip/route add dst-address=8.8.8.8 scope=10 gateway=10.111.0.1
/ip/route add dst-address=8.8.4.4 scope=10 gateway=10.112.0.1
/ip/route add distance=1 gateway=8.8.8.8 target-scope=11 check-gateway=ping
/ip/route add distance=2 gateway=8.8.4.4 target-scope=11 check-gateway=ping
```

[1] Static routes have scope 30 by default, which is why the host routes need `scope=10` to be usable as gateways. [1]

### NAT and open connections

Each WAN needs its own masquerade rule. [1] Connections open at the moment of failover can break. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Router | CHR in GNS3 (or hAP lite with two uplinks), `TODO` version |
| ISP1 | ether1, gateway `10.111.0.1` (CHR) |
| ISP2 | ether2, gateway `10.112.0.1` (CHR) |

## Method A — CLI

### Step 1 — Second WAN and NAT

```routeros
/interface/bridge/port remove [find interface=ether2]
/interface/list/member add list=WAN interface=ether2
/ip/firewall/nat add chain=srcnat action=masquerade out-interface=ether1
/ip/firewall/nat add chain=srcnat action=masquerade out-interface=ether2
```

[1] Add addresses for both links.

`TODO` ![Step 1](images/cli/01-wan2.png)

### Step 2 — Method 1 routes

Enter the two default routes with distances 1 and 2. [1]

`TODO` ![Step 2](images/cli/02-method1.png)

### Step 3 — Test Method 1

Disconnect ISP1's gateway. Within about 20 seconds the distance-2 route becomes active. [1]

```routeros
/ip/route/print where dst-address=0.0.0.0/0
```

`TODO` ![Step 3](images/cli/03-failover1.png)

### Step 4 — Method 2 routes

Remove the Method 1 routes and add the four Method 2 routes. [1] Now break ISP1's upstream instead of its gateway; failover still happens.

`TODO` ![Step 4](images/cli/04-method2.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Both up | Distance-1 route active | `TODO` |
| ISP1 gateway down | Distance-2 active (~20 s) [1] | `TODO` |
| ISP1 upstream down (Method 2) | Distance-2 active | `TODO` |

## Reset Before Method B

Remove the failover routes and the ether2 NAT rule; return ether2 to the bridge if needed.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Routes

**IP → Routes → +**: Gateway, Distance, Check Gateway `ping`; for Method 2, also Scope and Target Scope.

`TODO` ![Step 1](images/gui/01-routes.png)

### Step 2 — Test

Disconnect ISP1 and watch the route flags change.

`TODO` ![Step 2](images/gui/02-test.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| No failover when ISP1's internet fails | Only the gateway is checked | Method 2. [1] |
| Method 2 routes inactive | Host routes left at scope 30 | `scope=10`. [1] |
| Backup works, internet still fails | No NAT on ether2 | Step 1. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| Distance 1 / 2 + `check-gateway=ping` | Basic failover [1] |
| Host routes `scope=10` + `target-scope=11` | Check the internet, not the gateway [1] |
| One masquerade per WAN | NAT [1] |

## Key Takeaways

- Check something beyond the ISP's router to detect real outages.
- Expect existing connections to drop at the switch-over.

## References

1. [RouterOS Manual — Failover (WAN Backup)](https://manual.mikrotik.com/docs/high-availability-solutions/load-balancing/failover-wan-backup/)

---

**Previous:** [M.06 — IPsec Site-to-Site](../../M-vpn-and-network-tunneling/06-ipsec-site-to-site/README.md) · **Next:** N.02 — PCC Load Balancing

[Back to the main learning journal](../../README.md)
