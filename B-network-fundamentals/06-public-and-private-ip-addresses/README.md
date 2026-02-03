<div align="center">

# Public and Private IP Addresses

*Chapter B — Network Fundamentals & IP Addressing | Topic B.06*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Not every IPv4 address can be used on the internet. Some ranges are set aside for private networks, some for carrier networks, some for documentation, and some for special purposes inside a host or link. This topic lists the ranges a network technician meets every day and what each one means.

## Why This Matters

The MikroTik LAN `192.168.88.0/24` is a private range, which is exactly why the router needs NAT (B.10) to reach the internet. Recognising `169.254.x.x` on a laptop tells you DHCP failed; recognising `100.64.x.x` on the WAN tells you the ISP uses carrier-grade NAT and you cannot accept incoming connections directly.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.05 — Subnetting and CIDR](../05-subnetting-and-cidr/README.md) | Reading prefixes like `/12` and `/16`. |

## Theory

### Private address space

RFC 1918 reserves three blocks for private networks [1]:

| Block | Range | Prefix |
| --- | --- | --- |
| 10/8 | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| 172.16/12 | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| 192.168/16 | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

Private addresses have no global meaning. Routes to them must not be advertised between organisations, and packets with private source or destination addresses should not be forwarded across those links; ISPs are expected to filter them. [1]

### Why private addresses exist

They conserve globally unique address space and give an organisation flexible addressing without asking a registry. [1] The cost is that a host with a private address cannot talk to the internet directly. NAT (B.10) solves that; RFC 1918 notes that otherwise the host would need renumbering. [1]

### Shared address space for carrier-grade NAT

`100.64.0.0/10` is reserved as shared address space for the links between a provider's carrier-grade NAT (CGN) and customer equipment. [2] Like RFC 1918 space it is not globally routable, but it is meant for service-provider networks, and it must not be forwarded across provider boundaries. [2]

### Special-purpose ranges

RFC 6890 is the registry of special-purpose IPv4 blocks. [3] The ones you will see:

| Block | Name | Meaning |
| --- | --- | --- |
| 0.0.0.0/8 | "This host on this network" | Used only as a source while a host learns its address [3][4] |
| 127.0.0.0/8 | Loopback | Never appears outside a host [3][4] |
| 169.254.0.0/16 | Link local | Not forwarded by routers [3] |
| 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24 | Documentation (TEST-NET-1/2/3) | For examples in documents; should not appear on the internet [3][5] |

### Link-local addresses

A host may configure itself with an address in `169.254/16` when no other address configuration is available, such as on an isolated network. [6] Such an address only works with devices on the same link. [6] In practice, a laptop showing `169.254.x.x` did not get an address from a DHCP server.

### Public addresses

Everything outside the reserved blocks is globally routable public space, administered by IANA and the regional registries and assigned onward to ISPs. [7] Your ISP's router or CGN holds the public address your traffic appears to come from.

### How it connects

```text
Laptop 192.168.88.x (RFC 1918)
  └─ hAP lite: NAT ─► wlan1 address from home Wi-Fi (RFC 1918 again)
       └─ home router: NAT ─► WAN: public address, or 100.64.x.x if the ISP uses CGN (RFC 6598)
            └─ ISP ─► internet (public addresses only)
```

## How It Appears in RouterOS

```routeros
/ip/address/print
/ip/dhcp-client/print
```

Classify each address: the LAN bridge (`192.168.88.1/24`) and the wlan1 address from A.03 should both be private. [1]

> [!CAUTION]
> **Privacy:** if any address shown is public, cover it. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Address classes](images/cli/01-address-classes.png)

On the laptop, disconnect the cable from ether2, wait, and run `ipconfig` with the Ethernet adapter still enabled. If Windows assigns a `169.254.x.x` address, that is the link-local fallback. [6]

`TODO` ![Link-local](images/cli/02-link-local.png)

## Quick Recap

| Range | Meaning |
| --- | --- |
| 10/8, 172.16/12, 192.168/16 | Private, not routed on the internet [1] |
| 100.64/10 | Carrier-grade NAT shared space [2] |
| 127/8 | Loopback [3] |
| 169.254/16 | Link local; "no DHCP answer" on a laptop [3][6] |
| 192.0.2/24, 198.51.100/24, 203.0.113/24 | Documentation [5] |

## Key Takeaways

- Private addresses need NAT to reach the internet.
- `169.254.x.x` on a client points to a DHCP problem.
- `100.64.x.x` on the WAN means the ISP does NAT for you; port forwarding at home will not work through it.

## References

1. [RFC 1918 — Address Allocation for Private Internets](https://www.rfc-editor.org/rfc/rfc1918)
2. [RFC 6598 — IANA-Reserved IPv4 Prefix for Shared Address Space](https://www.rfc-editor.org/rfc/rfc6598)
3. [RFC 6890 — Special-Purpose IP Address Registries, §2.2.2](https://www.rfc-editor.org/rfc/rfc6890)
4. [RFC 1122 — Requirements for Internet Hosts, §3.2.1.3](https://www.rfc-editor.org/rfc/rfc1122)
5. [RFC 5737 — IPv4 Address Blocks Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc5737)
6. [RFC 3927 — Dynamic Configuration of IPv4 Link-Local Addresses](https://www.rfc-editor.org/rfc/rfc3927)
7. [RFC 4632 — Classless Inter-domain Routing (CIDR), §3.1](https://www.rfc-editor.org/rfc/rfc4632)

---

**Previous:** [B.05 — Subnetting and CIDR](../05-subnetting-and-cidr/README.md) · **Next:** B.07 — Gateway and Default Route

[Back to the main learning journal](../../README.md)
