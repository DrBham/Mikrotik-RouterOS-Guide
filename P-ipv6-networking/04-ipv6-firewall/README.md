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

The manual's IPv6 input chain uses an address list of allowed sources (link-local, your prefix, your ULA), accepts established/related traffic, drops invalid, accepts ICMPv6, traceroute probes (UDP 33434–33534), and DHCPv6 from `fe80::/10`, drops link-local sources arriving from the WAN, accepts the allowed list from the LAN, and drops everything else. [1] Method A Steps 1 and 2 give the full block.

### Protect the LAN (forward)

Accept established, related, and untracked; drop invalid; accept ICMPv6 from the LAN; accept new connections from the LAN, from the addresses of the `allowed` list; drop everything else. [1] This makes the LAN reachable only by connections it started. A LAN device whose address is not in the list can send only ICMPv6 to the internet. [1]

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

```routeros
/ipv6/firewall/address-list
add list=allowed address=fe80::/10 comment="Link-local"
add list=allowed address=2001:db8::/62 comment="Public prefix"
add list=allowed address=fd00:88::/64 comment="ULA prefix"
```

This is the manual's block. [1] Only the two prefixes are changed: the manual uses `2001:db8:1::/48` and `fd12:672e:6f65:8899::/64` and says to replace them with your prefixes. This lab uses the delegated prefix from P.03 (`2001:db8::/62`) and the ULA from P.02 (`fd00:88::/64`).

`TODO` ![Step 1](images/cli/01-list.png)

### Step 2 — Input chain

Enter the manual's IPv6 input rules in order. [1]

```routeros
/ipv6/firewall/filter
add chain=input action=accept \
    connection-state=established,related,untracked \
    comment="Accept established, related, untracked"
add chain=input action=drop connection-state=invalid \
    comment="Drop invalid"
add chain=input action=accept protocol=icmpv6 comment="Accept ICMPv6"
add chain=input action=accept protocol=udp dst-port=33434-33534 \
    comment="Accept UDP traceroute"
add chain=input action=accept protocol=udp dst-port=546 \
    src-address=fe80::/10 \
    comment="Accept DHCPv6 client prefix delegation"
add chain=input action=drop src-address=fe80::/10 \
    in-interface-list=WAN log=yes log-prefix=dropLL_from_public \
    comment="Drop link-local from the internet"
add chain=input action=accept src-address-list=allowed \
    in-interface-list=LAN \
    comment="Accept from the allowed addresses"
add chain=input action=drop comment="Drop everything else"
```

These lines are copied unchanged from the manual. [1]

`TODO` ![Step 2](images/cli/02-input.png)

### Step 3 — Forward chain

```routeros
/ipv6/firewall/filter
add chain=forward action=accept \
    connection-state=established,related,untracked \
    comment="Accept established, related, untracked"
add chain=forward action=drop connection-state=invalid \
    log=yes log-prefix=ipv6,invalid comment="Drop invalid"
add chain=forward action=accept protocol=icmpv6 \
    in-interface-list=LAN comment="Accept ICMPv6 from the LAN"
add chain=forward action=accept src-address-list=allowed \
    in-interface-list=LAN comment="Accept new connections from the LAN"
add chain=forward action=drop comment="Drop everything else"
```

These lines are copied unchanged from the manual's forward block. [1]

`TODO` ![Step 3](images/cli/03-forward.png)

### Step 4 — Test

From the WAN side (R1 in P.03), try to connect to a TCP port on a LAN host's global address: blocked. The forward chain accepts ICMPv6 only from the LAN, so a ping started on the WAN side is dropped too. [1] From the LAN, browsing over IPv6 works.

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

## Reset Before Method B

```routeros
/ipv6/firewall/filter/remove [find]
/ipv6/firewall/address-list/remove [find list=allowed]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Address list and rules

**IPv6 → Firewall → Address Lists** and **Filter Rules**, same order as Method A, in safe mode.

`TODO` ![Step 1](images/gui/01-rules.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| IPv6 stops working entirely | ICMPv6 dropped, breaking ND | Accept ICMPv6. [1] |
| Prefix lost after a while | DHCPv6 (UDP 546) dropped on input | Accept it from `fe80::/10`. [1] |
| Locked out | Management address not in `allowed` | Safe mode. [1] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| No NAT in IPv6 | Firewall is the only barrier |
| Accept ICMPv6 | ND depends on it [1] |
| Accept UDP 546 from `fe80::/10` | Prefix delegation [1] |
| Forward: new from LAN only | Default deny inbound [1] |

## Key Takeaways

- Every IPv6-enabled router needs an IPv6 firewall, not just an IPv4 one.
- Never drop all ICMPv6.

## References

1. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)

---

**Previous:** [P.03 — DHCPv6 Prefix Delegation](../03-dhcpv6-prefix-delegation/README.md) · **Next:** Chapter Q — Network Monitoring & Troubleshooting

[Back to the main learning journal](../../README.md)
