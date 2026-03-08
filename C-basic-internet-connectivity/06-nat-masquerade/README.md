<div align="center">

# NAT Masquerade

*Chapter C — Basic Internet Connectivity | Topic C.06*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The router is online and the LAN works, but the laptop still cannot reach the internet. One NAT rule fixes that. This topic adds it, watches it work, and closes the DNS hole that C.05 opened.

## Why This Matters

This is the last piece of a working home gateway. It also shows clearly that "the router has internet" and "the LAN has internet" are two different things.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.05 — LAN Bridge, IP, and DHCP Server](../05-lan-bridge-ip-and-dhcp-server/README.md) | A working LAN. |
| [B.10 — NAT Concepts](../../B-network-fundamentals/10-nat-concepts/README.md) | srcnat, masquerade, conntrack. |

## Theory

### Why the laptop has no internet yet

The laptop sends packets from `192.168.88.x`, a private address (B.06). The router forwards them out of the WAN, but the upstream network has no route back to `192.168.88.0/24`, so replies never return. Source NAT replaces the private source address with the router's WAN address. [1][2]

### The rule

The manual's masquerade rule:

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade src-address=10.0.0.0/24 out-interface-list=WAN comment="Masquerade the local network"
```

[1] `masquerade` suits a WAN whose address can change (DHCP, PPPoE), and its connections are cleared when the interface goes down. [1] `out-interface-list=WAN` means the rule follows whatever interface is in the `WAN` list, whether ether1, wlan1, or `pppoe-out1`. [1][3]

### Only the first packet

The rule matches only the first packet of each connection; connection tracking applies the translation to the rest. [1][4]

### DNS from the WAN

C.05 enabled `allow-remote-requests`, so the router answers DNS on every address, including the WAN. The DNS manual says to drop DNS from the WAN when you write your own firewall, or the router becomes an open resolver. [5] Its rules are added here; the rest of the firewall comes in Chapter H.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | C.02 + C.05 complete |

## Method A — CLI

### Step 1 — Confirm the problem

From the laptop:

```powershell
ping 192.168.88.1
ping 8.8.8.8
```

The first works, the second fails.

`TODO` ![Step 1](images/cli/01-no-internet.png)

### Step 2 — Add the masquerade rule

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade src-address=192.168.88.0/24 out-interface-list=WAN comment="Masquerade the local network"
/ip/firewall/nat/print
```

[1]

`TODO` ![Step 2](images/cli/02-masquerade.png)

### Step 3 — Test and watch the translations

```powershell
ping 8.8.8.8
ping google.com
```

```routeros
/ip/firewall/nat/print stats
/ip/firewall/connection/print count-only
```

[1][4] The rule's packet counter grows only with new connections.

`TODO` ![Step 3](images/cli/03-nat-working.png)

### Step 4 — Drop DNS requests from the WAN

```routeros
/ip/firewall/filter/add chain=input in-interface-list=WAN protocol=udp dst-port=53 action=drop comment="drop DNS from WAN"
/ip/firewall/filter/add chain=input in-interface-list=WAN protocol=tcp dst-port=53 action=drop comment="drop DNS from WAN"
```

[5]

`TODO` ![Step 4](images/cli/04-dns-drop.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Laptop ping 8.8.8.8 / google.com | Replies | `TODO` |
| Browser | This guide opens | `TODO` |
| NAT stats | Counters increase [1] | `TODO` |
| DNS drop rules | Present, input chain | `TODO` |

## Reset Before Method B

```routeros
/ip/firewall/nat/remove [find comment="Masquerade the local network"]
/ip/firewall/filter/remove [find comment="drop DNS from WAN"]
```

`TODO` ![Reset](images/gui/00-reset.png)

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Add the masquerade rule

**IP → Firewall → NAT → +**. General: Chain `srcnat`, Src. Address `192.168.88.0/24`, Out. Interface List `WAN`. Action: `masquerade`. Comment: `Masquerade the local network`.

`TODO` ![Step 1](images/gui/01-masquerade.png)

### Step 2 — Test

Ping and browse from the laptop. Watch the rule's **Packets** and **Bytes** columns.

`TODO` ![Step 2](images/gui/02-test.png)

### Step 3 — Drop DNS from the WAN

**IP → Firewall → Filter Rules → +** twice: Chain `input`, Protocol `udp` (then `tcp`), Dst. Port `53`, In. Interface List `WAN`, Action `drop`.

`TODO` ![Step 3](images/gui/03-dns-drop.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Rule counters stay at 0 | WAN interface not in the `WAN` list | `/interface/list/member/print`. [3] |
| Works with ether1 WAN, not after switching to wlan1 or PPPoE | New WAN interface not in the list | Add it to `WAN`. [3] |
| Ping by IP works, browser does not | DNS (C.05 Step 4) or MTU with PPPoE (C.04) | Check each. |
| Old connections still use a removed rule | Conntrack keeps existing translations [4] | `/ip/firewall/connection/remove [find]`. [4] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `chain=srcnat action=masquerade out-interface-list=WAN` | LAN to internet [1] |
| `src-address=192.168.88.0/24` | Translate only the LAN |
| `print stats` | Rule counters [1] |
| Drop UDP/TCP 53 from WAN | No open resolver [5] |

## Key Takeaways

- The router being online does not mean the LAN is; NAT connects the two.
- Use the `WAN` list so the rule survives a change of uplink.
- Every service you enable for the LAN must be blocked from the WAN.

## References

1. [RouterOS Manual — NAT](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/nat/)
2. [RFC 3022 — Traditional IP Network Address Translator, §2.2](https://www.rfc-editor.org/rfc/rfc3022)
3. [RouterOS CLI Reference — interface/list](https://manual.mikrotik.com/docs/cli-reference/interface/list/)
4. [RouterOS Manual — Connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/)
5. [RouterOS Manual — DNS](https://manual.mikrotik.com/docs/network-management/dns/)

---

**Previous:** [C.05 — LAN Bridge, IP, and DHCP Server](../05-lan-bridge-ip-and-dhcp-server/README.md) · **Next:** C.07 — Verify and Troubleshoot Connectivity

[Back to the main learning journal](../../README.md)
