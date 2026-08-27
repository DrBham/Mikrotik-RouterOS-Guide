<div align="center">

# IPv6 Firewall

*Chapter P — IPv6 Networking | Topic P.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

IPv6 devices usually have global addresses, with no NAT in front of them. The firewall is therefore the only thing deciding what the internet can reach. This topic builds the IPv6 input and forward chains from MikroTik's firewall guide.

## Why This Matters

An IPv4 LAN behind NAT is unreachable from outside by accident (B.10). An IPv6 LAN is reachable unless the firewall says otherwise, so forgetting the IPv6 firewall exposes every device.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.02 — Protecting the Router](../../H-firewall-and-network-security/02-protecting-the-router/README.md) | The same structure in IPv4. |
| [P.03 — DHCPv6 Prefix Delegation](../03-dhcpv6-prefix-delegation/README.md) | Global addresses to protect. |

## Theory

### What is different from IPv4

ICMPv6 carries Neighbor Discovery (P.02), so it cannot be dropped the way some IPv4 ICMP can; the manual's rules accept ICMPv6. [1] DHCPv6 prefix delegation arrives on UDP 546 from link-local addresses, which must be accepted on the WAN. [1]

### Protect the router (input)

The manual's IPv6 input chain uses an address list of allowed sources (link-local, your prefix, your ULA), accepts established/related traffic, drops invalid, accepts ICMPv6, traceroute probes (UDP 33434–33534), and DHCPv6 from `fe80::/10`, drops link-local sources arriving from the WAN, accepts the allowed list from the LAN, and drops everything else. [1]

```routeros
/ipv6/firewall/address-list add list=allowed address=fe80::/10 comment="Link-local"
/ipv6/firewall/filter add chain=input action=accept protocol=icmpv6 comment="Accept ICMPv6"
```

[1]

> [!WARNING]
> **Source needed:** only these two lines of the manual's IPv6 input block were captured verbatim during research. Copy the full block from the "Build an IPv6 firewall" section of the [Filter page](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/) into Method A before publishing.

### Protect the LAN (forward)

Accept established, related, and untracked; drop invalid; accept ICMPv6; accept new connections from the LAN; drop everything else. [1] This makes the LAN reachable only by connections it started.

### Lockout

The final drop rule can lock you out; keep your management address in `allowed` and its interface in `LAN`, or use safe mode. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | P.03 customer router, `TODO` version |
| LAN | Global prefix from P.03 |

## Method A — CLI

### Step 1 — Safe mode and address list

Press Ctrl+X, then add the `allowed` list entries for link-local, your delegated prefix, and your ULA. [1]

`TODO` ![Step 1](images/cli/01-list.png)

### Step 2 — Input chain

Enter the manual's IPv6 input rules in order. [1]

`TODO` ![Step 2](images/cli/02-input.png)

### Step 3 — Forward chain

```routeros
/ipv6/firewall/filter add chain=forward action=accept connection-state=established,related,untracked comment="Accept established, related, untracked"
/ipv6/firewall/filter add chain=forward action=drop connection-state=invalid comment="Drop invalid"
/ipv6/firewall/filter add chain=forward action=accept protocol=icmpv6 comment="Accept ICMPv6"
/ipv6/firewall/filter add chain=forward action=accept connection-state=new in-interface-list=LAN comment="Accept new from LAN"
/ipv6/firewall/filter add chain=forward action=drop comment="Drop everything else"
```

These follow the order the manual describes for the forward chain. [1] `TODO` compare each line with the manual's block.

`TODO` ![Step 3](images/cli/03-forward.png)

### Step 4 — Test

From the WAN side (R1 in P.03), ping a LAN host's global address: ICMPv6 passes. Try to connect to a TCP port on it: blocked. From the LAN, browsing over IPv6 works.

```routeros
/ipv6/firewall/filter/print stats
```

`TODO` ![Step 4](images/cli/04-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| LAN → internet over IPv6 | Works | `TODO` |
| WAN → LAN host TCP | Dropped | `TODO` |
| DHCPv6-PD after reboot | Still bound (UDP 546 accepted) | `TODO` |
