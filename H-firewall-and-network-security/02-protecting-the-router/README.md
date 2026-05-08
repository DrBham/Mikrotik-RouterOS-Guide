<div align="center">

# Protecting the Router

*Chapter H — Firewall & Network Security | Topic H.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Starting from the Chapter C router (empty configuration plus WAN, LAN, and NAT), this topic writes the `input` chain that protects the router itself, following the rules in MikroTik's firewall guide.

## Why This Matters

The Chapter C build has no input filter at all, so every service on the router is reachable from the WAN. This topic closes that gap and shows why the order of five short rules matters.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.01 — Firewall Concepts and Packet Flow](../01-firewall-concepts-and-packet-flow/README.md) | Chains, order, states. |
| [C.06 — NAT Masquerade](../../C-basic-internet-connectivity/06-nat-masquerade/README.md) | The router being protected. |
| [A.04 — RouterOS CLI Basics](../../A-mikrotik-fundamentals/04-routeros-cli-basics/README.md) | Safe mode. |

## Theory

### The manual's input rules

MikroTik's firewall guide protects the router with an address list of management addresses and five `input` rules [1]:

```routeros
/ip/firewall/address-list add list=allowed_to_router address=192.168.88.2-192.168.88.254
/ip/firewall/filter add chain=input action=accept connection-state=established,related,untracked comment="Accept established, related, untracked"
/ip/firewall/filter add chain=input action=drop connection-state=invalid comment="Drop invalid"
/ip/firewall/filter add chain=input action=accept src-address-list=allowed_to_router in-interface-list=LAN comment="Accept from the management addresses"
/ip/firewall/filter add chain=input action=accept protocol=icmp comment="Accept ICMP"
/ip/firewall/filter add chain=input action=drop comment="Drop everything else"
```

| Rule | Why |
| --- | --- |
| Accept established/related/untracked | Replies to the router's own connections (DNS, NTP, updates) [2] |
| Drop invalid | Packets that belong to nothing [2] |
| Accept from management addresses on LAN | Your access to WinBox, SSH, DNS |
| Accept ICMP | Ping and the ICMP errors that make the network work [1] |
| Drop everything else | Default deny |

### Lockout risk

The last rule drops everything not accepted above, including your own WinBox session if your address is not in the list or your interface is not in `LAN`. The manual warns about this and suggests safe mode. [1]

### DNS from the WAN

The final drop also covers DNS from the WAN, so the separate rules from C.06 become redundant but harmless. [3]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Chapter C build (C.02 + C.05 + C.06), lists `WAN` and `LAN` |
| Laptop | ether2, `192.168.88.x` |

## Method A — CLI

### Step 1 — Turn on safe mode

Press Ctrl+X. [4]

### Step 2 — Add the address list and rules

Enter the six commands from the theory section, in order. [1]

`TODO` ![Step 2](images/cli/02-rules.png)

### Step 3 — Check the order and counters

```routeros
/ip/firewall/filter/print stats
```

[1] The established rule should count most packets.

`TODO` ![Step 3](images/cli/03-stats.png)

### Step 4 — Leave safe mode

If WinBox and SSH still work, press Ctrl+X again to keep the changes. [4]

### Step 5 — Test from the WAN side

From a device on the upstream network, try to reach the router's WAN address on port 8291 (WinBox) or 22 (SSH). Both should fail, while ping succeeds.

```powershell
Test-NetConnection <router-WAN-address> -Port 8291
```

`TODO` ![Step 5](images/cli/05-wan-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| WinBox from LAN | Works | `TODO` |
| WinBox/SSH from WAN | Fails | `TODO` |
| Ping router from WAN | Works (ICMP accepted) | `TODO` |
| LAN internet | Still works (forward chain untouched) | `TODO` |

## Reset Before Method B

```routeros
/ip/firewall/filter/remove [find chain=input]
/ip/firewall/address-list/remove [find list=allowed_to_router]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Safe mode

Click **Safe Mode** in WinBox.

### Step 2 — Address list

**IP → Firewall → Address Lists → +**: Name `allowed_to_router`, Address `192.168.88.2-192.168.88.254`.

`TODO` ![Step 2](images/gui/02-address-list.png)

### Step 3 — Filter rules

**IP → Firewall → Filter Rules → +** five times, matching the CLI rules. Check the order in the list.

`TODO` ![Step 3](images/gui/03-rules.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli). Click **Safe Mode** again to keep the changes.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| WinBox disconnects after the last rule | Your address or interface not allowed | Safe mode undoes it; fix the list or `LAN` membership. [1] |
| Router cannot resolve names or update | Established rule missing or below the drop | Order matters. [1] |
| No rule counts anything | Rules in `forward` instead of `input` | H.01. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| Established/related first, invalid drop second | Standard start [1][2] |
| Address list for management | Who may reach the router [1] |
| Accept ICMP | Keep ping and errors working [1] |
| Final drop | Default deny for `input` [1] |
| Safe mode | Undo if you lock yourself out [4] |

## Key Takeaways

- Always work in safe mode when changing input rules.
- The router should answer management only from the LAN.

## References

1. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)
2. [RouterOS Manual — Connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/)
3. [RouterOS Manual — DNS](https://manual.mikrotik.com/docs/network-management/dns/)
4. [RouterOS Manual — Console](https://manual.mikrotik.com/docs/management-tools/console/)

---

**Previous:** [H.01 — Firewall Concepts and Packet Flow](../01-firewall-concepts-and-packet-flow/README.md) · **Next:** H.03 — Protecting the LAN

[Back to the main learning journal](../../README.md)
