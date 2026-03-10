<div align="center">

# Verify and Troubleshoot Connectivity

*Chapter C — Basic Internet Connectivity | Topic C.07*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The gateway from C.02–C.06 works. This topic teaches how to prove it, and how to find the fault when it does not, using three RouterOS tools: ping, traceroute, and torch. The practice part breaks the lab on purpose, one layer at a time, and finds each fault.

## Why This Matters

"No internet" is the most common complaint in networking. A fixed order of tests turns it from guessing into a short checklist, and each test points at one layer from Chapter B.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.06 — NAT Masquerade](../06-nat-masquerade/README.md) | A complete gateway to test. |
| [B.07 — Gateway and Default Route](../../B-network-fundamentals/07-gateway-and-default-route/README.md) | TTL and unreachable messages. |

## Theory

### Ping

Ping sends ICMP echo requests and shows each reply with its sequence number, size, TTL, and time, then a summary of sent, received, loss, and round-trip times. [1] The status column reports errors such as timeout, net unreachable, host unreachable, and TTL exceeded. [1] Useful options: `count`, `size`, `do-not-fragment`, `src-address`, `interface`, and `arp-ping=yes` to test layer 2 reachability. [1]

### Traceroute

Traceroute lists the routers on the path, with loss and round-trip times per hop. [2] It sends probes with TTL 1, 2, 3, and so on; each router that drops a probe at TTL zero returns ICMP time exceeded. [2] RouterOS uses ICMP probes by default and stops at 30 hops. [2] The status column shows errors such as host or network unreachable, packet filtered, or fragmentation needed. [2]

### Torch

Torch shows live traffic on one interface, grouped by addresses, protocol, and ports, with transmit and receive rates. [3] It sees packets before the firewall, so traffic that the firewall drops still appears on the interface where it arrived. [3]

### A test order that follows the layers

| # | Test | Layer | If it fails, look at |
| --- | --- | --- | --- |
| 1 | Laptop has `192.168.88.x` and gateway `.1` | Link + DHCP | Cable, bridge port, DHCP server (C.05) |
| 2 | Laptop pings `192.168.88.1` | Internet, local | Address, mask (B.04) |
| 3 | Router pings its WAN gateway | Internet, WAN | WAN address, link (C.02–C.04) |
| 4 | Router pings `8.8.8.8` | Routing | Default route (B.07) |
| 5 | Router pings `google.com` | DNS | Router DNS servers (B.08) |
| 6 | Laptop pings `8.8.8.8` | NAT | Masquerade and `WAN` list (C.06) |
| 7 | Laptop pings `google.com` | DNS for LAN | `allow-remote-requests`, DHCP DNS option (C.05) |

The first failing test names the broken layer.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | C.02 + C.05 + C.06 complete and working |

## Method A — CLI

### Step 1 — Baseline: all tests pass

Run the seven tests from the table. On the router:

```routeros
/ping 8.8.8.8 count=3
/ping google.com count=3
/tool/traceroute 8.8.8.8 count=1
```

[1][2]

> [!CAUTION]
> **Privacy:** traceroute shows your provider's routers and may show your public address. Cover the hops after your own upstream router. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-baseline.png)

### Step 2 — Watch traffic with torch

While the laptop pings `8.8.8.8` continuously (`ping -t 8.8.8.8`):

```routeros
/tool/torch bridge-lan
```

[3] Stop with Ctrl+C. Then run it on the WAN interface and compare the source address: on the WAN side it is the router's address, because of NAT.

`TODO` ![Step 2](images/cli/02-torch.png)

### Step 3 — Break DNS

```routeros
/ip/dns/set allow-remote-requests=no
```

Re-run tests 5–7: 5 and 6 pass, 7 fails. Restore with `allow-remote-requests=yes`. [4]

`TODO` ![Step 3](images/cli/03-break-dns.png)

### Step 4 — Break NAT

```routeros
/ip/firewall/nat/disable [find comment="Masquerade the local network"]
/ip/firewall/connection/remove [find]
```

Tests 1–5 pass, 6 fails. Torch on the WAN now shows the laptop's private source address leaving the router. Re-enable the rule. [5][6]

`TODO` ![Step 4](images/cli/04-break-nat.png)

### Step 5 — Break the default route

For a DHCP WAN:

```routeros
/ip/dhcp-client/set [find] add-default-route=no
```

Test 3 passes, test 4 fails with no reply or network unreachable. Set it back to `yes`. [7]

`TODO` ![Step 5](images/cli/05-break-route.png)

### Verify (CLI)

| Fault | First failing test | Actual result |
| --- | --- | --- |
| DNS off for LAN | 7 | `TODO` |
| NAT disabled | 6 | `TODO` |
| No default route | 4 | `TODO` |
| All restored | None | `TODO` |
