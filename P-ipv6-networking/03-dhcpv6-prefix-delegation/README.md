<div align="center">

# DHCPv6 Prefix Delegation

*Chapter P — IPv6 Networking | Topic P.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

ISPs that offer IPv6 usually delegate a whole prefix to the customer router with DHCPv6. The router keeps it in a pool and carves `/64` subnets for its LANs, which clients then configure with SLAAC. This topic builds that chain, with a CHR playing the ISP.

## Why This Matters

This is how real home and office IPv6 works. Without prefix delegation, the router has nothing to advertise to its LAN.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [P.02 — Neighbor Discovery and SLAAC](../02-neighbor-discovery-and-slaac/README.md) | Advertising a `/64`. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | DHCP in IPv4. |

## Theory

### Prefix delegation

DHCPv6 provides a mechanism to delegate IPv6 prefixes automatically. [1] With `request=prefix`, the RouterOS DHCPv6 client puts the delegated prefix into a pool named by `pool-name`. [2][3] `pool-prefix-length` sets the size of the pieces taken from it; it must be equal to or longer than the received prefix. [3]

### Using the pool

An address with `from-pool` takes a subnet from the pool: `::1/64` becomes `<prefix>::1/64`. [4] With `advertise=yes`, LAN clients use it for SLAAC (P.02). [4]

### Default route

DHCPv6 itself does not provide a gateway; `add-default-route=yes` on the DHCPv6 client adds `::/0` towards the DHCPv6 server, and it is off by default. [3] Often the ISP's router advertisement provides the default route instead.

### The manual's example

ISP side (R1): [2]

```routeros
/ipv6/pool add name=myPool prefix=2001:db8::/62 prefix-length=64
/ipv6/dhcp-server add prefix-pool=myPool disabled=no interface=to-CE-routers lease-time=3m name=server1
```

Customer side (CE): [2]

```routeros
/ipv6/dhcp-client add interface=to-R1 request=prefix pool-name=my-ipv6
/ipv6/address add address=::1/64 from-pool=my-ipv6 interface=to-clients advertise=yes
```

## Lab Environment

| Item | Value |
| --- | --- |
| ISP | CHR R1, interface `to-CE-routers`, `TODO` version |
| Customer | hAP lite or CHR, WAN `to-R1` (ether1), LAN `to-clients` (bridge) |
| Prefix | `2001:db8::/62` (documentation) |

Rename interfaces to match the manual, or adapt the names.

## Method A — CLI

### Step 1 — ISP side

On R1, the two ISP commands. [2]

`TODO` ![Step 1](images/cli/01-isp.png)

### Step 2 — DHCPv6 client

On the customer router:

```routeros
/ipv6/dhcp-client add interface=to-R1 request=prefix pool-name=my-ipv6
/ipv6/dhcp-client print
/ipv6/pool print
```

[2] Status `bound` and a dynamic pool `my-ipv6`.

`TODO` ![Step 2](images/cli/02-client.png)

### Step 3 — LAN address from the pool

```routeros
/ipv6/address add address=::1/64 from-pool=my-ipv6 interface=to-clients advertise=yes
/ipv6/address print
```

[2][4]

`TODO` ![Step 3](images/cli/03-address.png)

### Step 4 — Check on R1 and the laptop

On R1:

```routeros
/ipv6/dhcp-server/binding print
```

[2] On the laptop, `ipconfig` shows a `2001:db8:…` address.

`TODO` ![Step 4](images/cli/04-check.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| DHCPv6 client | `bound` [3] | `TODO` |
| Pool | Dynamic `my-ipv6` | `TODO` |
| LAN address | `2001:db8:…::1/64` | `TODO` |
| Laptop | Global address by SLAAC | `TODO` |

## Reset Before Method B

```routeros
/ipv6/address/remove [find from-pool=my-ipv6]
/ipv6/dhcp-client/remove [find]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — DHCPv6 client

**IPv6 → DHCP Client → +**: Interface `to-R1`, Request `prefix`, Pool Name `my-ipv6`.

`TODO` ![Step 1](images/gui/01-client.png)

### Step 2 — Address from pool

**IPv6 → Addresses → +**: Address `::1/64`, From Pool `my-ipv6`, Interface `to-clients`, Advertise.

`TODO` ![Step 2](images/gui/02-address.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Client stays `searching...` | No DHCPv6 server, or WAN firewall drops UDP 546 | Allow DHCPv6 from `fe80::/10` (P.04). [5] |
| Pool empty | `request=address` instead of `prefix` | Use `prefix`. [3] |
| LAN has addresses, no internet | No IPv6 default route on the router | `add-default-route=yes` or ISP RA. [3] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `request=prefix pool-name=` | Receive a delegated prefix [3] |
| `from-pool=` + `::1/64` | Make a LAN subnet [4] |
| `advertise=yes` | Clients use SLAAC [4] |
| `add-default-route` | Off by default [3] |

## Key Takeaways

- The ISP delegates a prefix; the router splits it into `/64` LANs.
- Global IPv6 addresses reveal your prefix; cover them in screenshots.

## References

1. [RFC 8415 — Dynamic Host Configuration Protocol for IPv6, §1](https://www.rfc-editor.org/rfc/rfc8415)
2. [RouterOS Manual — DHCPv6 Client](https://manual.mikrotik.com/docs/network-management/dhcp/dhcpv6-client/)
3. [RouterOS CLI Reference — ipv6/dhcp-client](https://manual.mikrotik.com/docs/cli-reference/ipv6/dhcp-client/)
4. [RouterOS CLI Reference — ipv6/address](https://manual.mikrotik.com/docs/cli-reference/ipv6/address/)
5. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)

---

**Previous:** [P.02 — Neighbor Discovery and SLAAC](../02-neighbor-discovery-and-slaac/README.md) · **Next:** P.04 — IPv6 Firewall

[Back to the main learning journal](../../README.md)
