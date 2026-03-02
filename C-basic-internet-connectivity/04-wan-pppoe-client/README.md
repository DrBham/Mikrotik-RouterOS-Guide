<div align="center">

# WAN via PPPoE Client

*Chapter C — Basic Internet Connectivity | Topic C.04*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Many fibre and DSL providers authenticate customers with PPPoE: the router logs in with a user name and password and receives its address that way. This topic configures a RouterOS PPPoE client. Because most learners have no PPPoE line to practise on, the lab also builds a small PPPoE "ISP" on a Cloud Hosted Router (CHR).

## Why This Matters

When a provider's modem is set to bridge mode, the MikroTik must dial PPPoE itself. PPPoE also changes two things you learned earlier: the WAN interface is a new virtual interface, not ether1, and its MTU is smaller than 1500.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.02 — WAN via DHCP Client](../02-wan-dhcp-client/README.md) | WAN list and testing. |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | Ethernet MTU 1500. |

## Theory

### PPP over Ethernet

PPPoE carries PPP sessions over Ethernet (RFC 2516), and RouterOS has both a client and a server. [1] A session starts with a discovery phase (PADI, PADO, PADR, PADS) and then runs PPP: LCP, authentication with PAP, CHAP, or MS-CHAP, and IPCP to get an address. [1]

### The client

| Property | Effect |
| --- | --- |
| `interface` | The Ethernet port facing the provider. [2] |
| `user`, `password` | The account from the provider. [2] |
| `add-default-route=yes` | Install a default route over the session. [1][2] |
| `use-peer-dns=yes` | Use the provider's DNS servers. [1][2] |
| `service-name` | Blank connects to any server. [2] |

The PPPoE session is its own interface (for example `pppoe-out1`), and that interface, not ether1, goes into the `WAN` list so NAT and the firewall use it. [1]

### MTU 1492

PPPoE uses 6 bytes of each Ethernet frame and PPP another 2, so a PPPoE link has an MTU of 1492 bytes. [1] The client's `max-mtu` and `max-mru` set to `auto` give 1492 on a 1500-byte Ethernet link. [2] Slow web pages while ping still works is a typical MTU symptom; MSS clamping in the PPP profiles is the fix the manual describes. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Client | MikroTik hAP lite, empty configuration + `WAN`/`LAN` lists (C.02 Steps 1, 2) |
| "ISP" | MikroTik CHR in GNS3 or VirtualBox, `TODO` RouterOS version |
| Link | hAP lite ether1 ↔ CHR ether3 (bridged to the physical NIC) |
| Account | user `alice`, password `Example-pass-1` (from the manual's example) |

> [!NOTE]
> `TODO` document how the CHR's ether3 is bridged to the laptop's second NIC or a USB Ethernet adapter, or run both routers as CHRs in GNS3.

### ISP side (CHR)

These are the manual's server commands. [1]

```routeros
/ip/pool/add name=pppoe-pool ranges=10.0.0.10-10.0.0.100
/ppp/profile/add name=subscribers local-address=10.0.0.1 remote-address=pppoe-pool
/ppp/secret/add name=alice password=Example-pass-1 profile=subscribers service=pppoe
/interface/pppoe-server/server/add service-name=internet interface=ether3 default-profile=subscribers disabled=no
```

Do not give the server interface an IP address or a DHCP server. [1]

`TODO` ![ISP side](images/cli/00-isp-server.png)

## Method A — CLI

### Step 1 — Scan for PPPoE servers

```routeros
/interface/pppoe-client/scan ether1 duration=5
```

[1] The `internet` service from the CHR should appear.

`TODO` ![Step 1](images/cli/01-scan.png)

### Step 2 — Add the client

```routeros
/interface/pppoe-client/add name=pppoe-out1 interface=ether1 user=alice password=Example-pass-1 add-default-route=yes use-peer-dns=yes disabled=no
```

[1]

> [!CAUTION]
> **Privacy:** with a real ISP, cover the user name and password. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-client-add.png)

### Step 3 — Check the session

```routeros
/interface/pppoe-client/monitor pppoe-out1 once
```

Expect `status: connected` and `mtu: 1492`. [1]

`TODO` ![Step 3](images/cli/03-monitor.png)

### Step 4 — Make it the WAN

```routeros
/interface/list/member/add list=WAN interface=pppoe-out1
/ip/address/print
/ip/route/print
```

[1] The address from the pool and a default route via `pppoe-out1` appear.

`TODO` ![Step 4](images/cli/04-wan-list.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Monitor | `connected`, MTU 1492 [1] | `TODO` |
| Address | From `10.0.0.10–100` | `TODO` |
| Default route | Via `pppoe-out1` | `TODO` |
| Ping CHR `10.0.0.1` | Replies | `TODO` |

## Reset Before Method B

```routeros
/interface/pppoe-client/remove pppoe-out1
/interface/list/member/remove [find interface=pppoe-out1]
```

`TODO` ![Reset](images/gui/00-reset.png)

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Add the client

**PPP → Interface → + → PPPoE Client**. General: Interface `ether1`. Dial Out: User `alice`, Password, tick **Use Peer DNS** and **Add Default Route**. **OK**.

`TODO` ![Step 1](images/gui/01-client.png)

### Step 2 — Check the status

Open the client and look at **Status**: connected, MTU 1492.

`TODO` ![Step 2](images/gui/02-status.png)

### Step 3 — Make it the WAN

**Interfaces → Interface List → +**: List `WAN`, Interface `pppoe-out1`.

`TODO` ![Step 3](images/gui/03-wan-list.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Client stays disconnected | No server reachable on that port | Run the scan. [1] |
| Authentication fails | Wrong user or password | Check `/ppp/secret` on the server. |
| Ping works, some websites hang | MTU 1492 | MSS clamping (`change-tcp-mss`) in the PPP profile. [1] |
| LAN has no internet after switching to PPPoE | NAT still points at ether1 | Put `pppoe-out1` in the `WAN` list. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/interface/pppoe-client/add …` | Dial PPPoE [1] |
| `monitor … once` | Status and MTU [1] |
| MTU 1492 | 1500 minus PPPoE 6 and PPP 2 [1] |
| WAN list | `pppoe-out1`, not ether1 [1] |
| `scan` | Find PPPoE servers [1] |

## Key Takeaways

- With PPPoE, the WAN is the virtual `pppoe-out1` interface.
- The PPPoE MTU is 1492, which explains a class of "half-working" internet problems.
- A CHR makes a good practice ISP.

## References

1. [RouterOS Manual — PPPoE](https://manual.mikrotik.com/docs/virtual-private-networks/pppoe/)
2. [RouterOS CLI Reference — interface/pppoe-client](https://manual.mikrotik.com/docs/cli-reference/interface/pppoe-client/)

---

**Previous:** [C.03 — WAN via Static IP, Route, and DNS](../03-wan-static-ip-route-and-dns/README.md) · **Next:** C.05 — LAN Bridge, IP, and DHCP Server

[Back to the main learning journal](../../README.md)
