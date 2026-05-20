<div align="center">

# Address Lists and RAW Rules

*Chapter H — Firewall & Network Security | Topic H.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Address lists let one rule match many addresses, and rules can add addresses to a list automatically. RAW rules run before connection tracking, so they can drop unwanted traffic cheaply. This topic combines both: it catches hosts probing a port and blocks them for a day.

## Why This Matters

Every internet-facing router is scanned constantly. Dropping known bad sources in RAW, before the router spends effort tracking their connections, is cheaper than dropping them in the filter.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.02 — Protecting the Router](../02-protecting-the-router/README.md) | Input chain and address lists. |
| [H.01 — Firewall Concepts and Packet Flow](../01-firewall-concepts-and-packet-flow/README.md) | RAW before connection tracking. |

## Theory

### Address lists

An address list is a named set of addresses or ranges under `/ip/firewall/address-list`. [1] Static entries are saved in the configuration; entries with a `timeout` are dynamic, marked `D`, and are lost on reboot. [1] The longest timeout is `35w3d13h13m56s`. [1]

```routeros
/ip/firewall/address-list/add list=blocked address=203.0.113.0/24 comment="Abuse reports"
```

[1]

### Adding addresses from a rule

The action `add-src-to-address-list` puts the source of a matching packet into a list, with an optional `address-list-timeout`. [1] The manual's example lists hosts that probe the Telnet port from the WAN, then drops everything from them [1]:

```routeros
/ip/firewall/raw/add chain=prerouting action=add-src-to-address-list in-interface-list=WAN protocol=tcp dst-port=23 src-address-list=!trusted address-list=port-scanners address-list-timeout=1d comment="List hosts that probe the Telnet port"
/ip/firewall/raw/add chain=prerouting action=drop in-interface-list=WAN src-address-list=port-scanners comment="Drop listed hosts"
```

### RAW

RAW has only the `prerouting` and `output` chains and runs before connection tracking. [2] It can `drop` early or mark traffic `notrack` to skip tracking; untracked traffic cannot be NATed, and the filter must accept the `untracked` state (the default configuration already does). [2]

### How it connects

```text
H.04 lists + RAW ──► Chapter I restrict port forwards to a source list ──► Chapter M allow VPN only from known peers
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | H.02 + H.03 complete |
| Test host | A device on the upstream (WAN) network |

## Method A — CLI

### Step 1 — A trusted list

```routeros
/ip/firewall/address-list/add list=trusted address=192.168.88.0/24
```

[1] The detection rule skips sources in `trusted`.

`TODO` ![Step 1](images/cli/01-trusted.png)

### Step 2 — Detection and drop rules

Enter the two RAW rules from the theory section. [1]

`TODO` ![Step 2](images/cli/02-raw.png)

### Step 3 — Trigger it

From the WAN-side device:

```powershell
Test-NetConnection <router-WAN-address> -Port 23
ping <router-WAN-address>
```

The port test adds the device to `port-scanners`; after that even ping fails.

`TODO` ![Step 3](images/cli/03-trigger.png)

### Step 4 — Inspect the list

```routeros
/ip/firewall/address-list/print where list=port-scanners
/ip/firewall/raw/print stats
```

[1]

> [!CAUTION]
> **Privacy:** cover public addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 4](images/cli/04-list.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| After port 23 probe | Source in `port-scanners`, `D` flag, ~1d timeout [1] | `TODO` |
| Ping from that host | Dropped | `TODO` |
| LAN traffic | Unaffected | `TODO` |

## Reset Before Method B

```routeros
/ip/firewall/raw/remove [find]
/ip/firewall/address-list/remove [find list=port-scanners]
/ip/firewall/address-list/remove [find list=trusted]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Lists

**IP → Firewall → Address Lists → +**: `trusted`, `192.168.88.0/24`.

`TODO` ![Step 1](images/gui/01-list.png)

### Step 2 — RAW rules

**IP → Firewall → Raw → +**: the two rules, with Action `add src to address list` and `drop`.

`TODO` ![Step 2](images/gui/02-raw.png)

### Verify (WinBox)

Trigger from the WAN side and watch **Address Lists** fill.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| You blocked yourself | Your address not in `trusted` | Remove it from the list; add it to `trusted`. [1] |
| List empty after reboot | Dynamic entries are not saved [1] | Expected. |
| NAT broken for some traffic | `notrack` used on NATed traffic [2] | Do not notrack traffic that needs NAT. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/ip/firewall/address-list` | Named address sets [1] |
| `add-src-to-address-list` + timeout | Learn bad sources automatically [1] |
| RAW `prerouting` | Drop before connection tracking [2] |
| `notrack` | Skip tracking; no NAT [2] |

## Key Takeaways

- Lists turn many rules into one.
- RAW is the cheapest place to drop traffic you never want.

## References

1. [RouterOS Manual — Address Lists](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/address-lists/)
2. [RouterOS Manual — RAW](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/raw/)

---

**Previous:** [H.03 — Protecting the LAN](../03-protecting-the-lan/README.md) · **Next:** Chapter I — Network Address Translation (NAT)

[Back to the main learning journal](../../README.md)
