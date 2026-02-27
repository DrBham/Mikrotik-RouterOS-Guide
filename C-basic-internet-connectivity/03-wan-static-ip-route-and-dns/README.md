<div align="center">

# WAN via Static IP, Route, and DNS

*Chapter C — Basic Internet Connectivity | Topic C.03*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Some ISPs give a fixed address instead of DHCP. Then the three things a DHCP client did automatically in C.02 must be set by hand: the address, the default route, and DNS. This topic does exactly that.

## Why This Matters

Business lines, data-centre uplinks, and links between routers often use static addresses. Doing it by hand also proves you understand what DHCP delivered: if any of the three pieces is missing or wrong, the symptom tells you which one.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.02 — WAN via DHCP Client](../02-wan-dhcp-client/README.md) | Same lab, same lists. |
| [B.07 — Gateway and Default Route](../../B-network-fundamentals/07-gateway-and-default-route/README.md) | The default route. |
| [B.08 — DNS Concepts](../../B-network-fundamentals/08-dns-concepts/README.md) | DNS servers. |

## Theory

### The three pieces

The official guide configures a static WAN with three commands [1]:

```routeros
/ip/address/add address=1.2.3.100/24 interface=ether1
/ip/route/add gateway=1.2.3.1
/ip/dns/set servers=8.8.8.8
```

| Command | Replaces from DHCP | Theory |
| --- | --- | --- |
| `/ip/address/add` | The leased address | B.04 |
| `/ip/route/add gateway=…` | `add-default-route` | B.07 |
| `/ip/dns/set servers=…` | `use-peer-dns` | B.08 |

A route added with only a `gateway` uses the default destination `0.0.0.0/0`, as the manual's example `add gateway=10.155.125.1` shows. [2]

### Values come from the provider

The ISP gives the address, prefix, gateway, and DNS servers. In this lab the "provider" is your upstream network, so pick an address in its subnet that its DHCP server will not hand to anyone else.

> [!WARNING]
> A static address that is also in the upstream DHCP pool can be given to another device, which causes an address conflict. Check the upstream router's DHCP range first.

### Watching the gateway

With a static default route, RouterOS keeps using it even if the gateway stops answering, unless `check-gateway` is set. With `check-gateway` the router verifies the gateway with ARP, ICMP, or BFD and stops using the route when it fails. [2] Chapter N builds failover on this.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| WAN | `TODO` ether1 or wlan1 |
| Upstream subnet | `TODO` e.g. 192.168.1.0/24, gateway 192.168.1.1 |
| Chosen static address | `TODO` outside the upstream DHCP pool |
| Starting state | Empty configuration + `WAN`/`LAN` lists (C.02 Steps 1–3, 5) |

The examples below use `192.168.1.50/24` and gateway `192.168.1.1`. Replace them with your values.

## Method A — CLI

### Step 1 — Prepare

Reset to an empty configuration, create the lists, prepare wlan1 if used, and add the WAN interface to the `WAN` list, as in C.02 Steps 1, 2, 3, and 5. Do not add a DHCP client.

`TODO` ![Step 1](images/cli/01-prepare.png)

### Step 2 — Add the WAN address

```routeros
/ip/address/add address=192.168.1.50/24 interface=ether1
/ip/address/print
```

[1][3]

`TODO` ![Step 2](images/cli/02-address.png)

### Step 3 — Test the gateway before routing

```routeros
/ping 192.168.1.1 count=3
```

The gateway is on the connected network, so this works without any route. [4]

`TODO` ![Step 3](images/cli/03-ping-gateway.png)

### Step 4 — Add the default route

```routeros
/ip/route/add gateway=192.168.1.1
/ip/route/print
/ping 8.8.8.8 count=3
```

[1][2]

`TODO` ![Step 4](images/cli/04-default-route.png)

### Step 5 — Set DNS

```routeros
/ip/dns/set servers=8.8.8.8
/ping google.com count=3
```

[1][5]

`TODO` ![Step 5](images/cli/05-dns.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Ping gateway | Replies (Step 3) | `TODO` |
| Ping 8.8.8.8 | Replies only after Step 4 | `TODO` |
| Ping google.com | Replies only after Step 5 | `TODO` |
| `/ip/route/print` | Static `0.0.0.0/0` via your gateway | `TODO` |

## Reset Before Method B

```routeros
/system/reset-configuration no-defaults=yes skip-backup=yes
```

Repeat Method A Step 1 preparation (lists and WAN membership) in WinBox or the terminal.

`TODO` ![Reset](images/gui/00-reset.png)

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Add the WAN address

**IP → Addresses → +**: Address `192.168.1.50/24`, Interface `ether1`.

`TODO` ![Step 1](images/gui/01-address.png)

### Step 2 — Add the default route

**IP → Routes → +**: Dst. Address `0.0.0.0/0`, Gateway `192.168.1.1`.

`TODO` ![Step 2](images/gui/02-default-route.png)

### Step 3 — Set DNS

**IP → DNS**: Servers `8.8.8.8`.

`TODO` ![Step 3](images/gui/03-dns.png)

### Step 4 — Test

**Tools → Ping**: the gateway, `8.8.8.8`, then `google.com`.

`TODO` ![Step 4](images/gui/04-ping.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Gateway does not answer | Wrong prefix or address, or link down | Compare with the upstream subnet (B.04). |
| Gateway answers, 8.8.8.8 does not | No default route, or gateway typo | `/ip/route/print`. [2] |
| 8.8.8.8 answers, names fail | No DNS server | `/ip/dns/print`. [5] |
| Works for a while, then another device loses access | Static address inside upstream DHCP pool | Choose an address outside the pool. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/ip/address/add address=a/p interface=wan` | Static WAN address [1] |
| `/ip/route/add gateway=g` | Default route [1][2] |
| `/ip/dns/set servers=d` | Router DNS [1][5] |
| `check-gateway` | Stop using a dead gateway [2] |

## Key Takeaways

- Static WAN = address + default route + DNS; test after each one.
- The symptom points to the missing piece: no gateway reply, no internet IP, or no names.
- Never pick a static address from another device's DHCP pool.

## References

1. [RouterOS Manual — First Time Configuration](https://manual.mikrotik.com/docs/getting-started/first-time-configuration)
2. [RouterOS Manual — Routing decision](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/routing-decision/)
3. [RouterOS CLI Reference — ip/address](https://manual.mikrotik.com/docs/cli-reference/ip/address/)
4. [RFC 1122 — Requirements for Internet Hosts, §3.3.1](https://www.rfc-editor.org/rfc/rfc1122)
5. [RouterOS Manual — DNS](https://manual.mikrotik.com/docs/network-management/dns/)

---

**Previous:** [C.02 — WAN via DHCP Client](../02-wan-dhcp-client/README.md) · **Next:** C.04 — WAN via PPPoE Client

[Back to the main learning journal](../../README.md)
