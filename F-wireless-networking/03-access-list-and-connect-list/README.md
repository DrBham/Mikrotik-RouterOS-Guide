<div align="center">

# Access List and Connect List

*Chapter F — Wireless Networking | Topic F.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The access list decides which clients an AP accepts; the connect list decides which APs a station may join. This topic uses the access list to reject weak-signal clients and to block one device, and the connect list to pin the A.03 uplink to one AP.

## Why This Matters

A client at the edge of coverage slows the whole AP down, and a station that roams to the wrong network breaks the uplink. Both lists give control beyond the shared password.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [F.02 — Wireless Security Profiles](../02-wireless-security-profiles/README.md) | A secured AP. |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | Station mode for the connect list. |

## Theory

### Default authentication

On an AP, `default-authentication` decides what happens to clients that match no access-list entry; on a station, it decides whether to connect to APs that match no connect-list entry. [1]

### Access list (AP side)

Entries are checked in order and the first match applies. [1] An entry can match a MAC address, an interface, or a signal range, and allow or deny authentication and forwarding. [1] Example from the manual's description:

```routeros
/interface/wireless/access-list add authentication=yes forwarding=yes interface=wlan2 signal-range=-55..0
```

### Connect list (station side)

The connect list chooses which APs a station may join. [1] Examples from the manual's description:

```routeros
/interface/wireless/set station-wlan default-authentication=no
/interface/wireless/connect-list/add interface=station-wlan connect=yes mac-address=00:11:22:33:00:01
```

> [!WARNING]
> **Source needed:** the access-list and connect-list examples come from a summary of the Wireless Interface page, not verified text. Confirm them on the [Wireless Interface page](https://manual.mikrotik.com/docs/wireless/abgn/wireless-interface/) before publishing.

### MAC filtering is not security

A MAC address can be copied, so the manual's HotSpot guide warns to use MAC-based access only for devices that cannot do better. [2] Use the access list for signal and policy control, and WPA2 (F.02) for security.

### How it connects

```text
F.03 access list ──► Chapter L HotSpot IP bindings use the same "allow by MAC" idea, with the same caveat
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | F.02 Method A complete |
| Clients | Two phones (or a phone and a laptop) |

## Method A — CLI

### Step 1 — Reject weak clients

```routeros
/interface/wireless/access-list add interface=wlan1 signal-range=-75..0 authentication=yes forwarding=yes comment="good signal"
/interface/wireless/set wlan1 default-authentication=no
```

[1] Walk away with a phone until its signal drops below −75 dBm and watch it disconnect.

`TODO` ![Step 1](images/cli/01-signal-range.png)

### Step 2 — Block one device

```routeros
/interface/wireless/access-list add interface=wlan1 mac-address=00:00:5E:00:53:20 authentication=no place-before=0 comment="blocked device"
```

[1] Use the second phone's MAC (the example is a documentation address). It can no longer join.

> [!CAUTION]
> **Privacy:** cover real MAC addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-block.png)

### Step 3 — Pin a station to one AP (optional)

On a router running the A.03 station uplink:

```routeros
/interface/wireless/connect-list/add interface=wlan1 connect=yes mac-address=00:00:5E:00:53:01
/interface/wireless/set wlan1 default-authentication=no
```

[1] Use your upstream AP's MAC from the scan.

`TODO` ![Step 3](images/cli/03-connect-list.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Weak phone | Disconnected below −75 dBm | `TODO` |
| Blocked phone | Cannot join | `TODO` |
| Station | Joins only the listed AP | `TODO` |

## Reset Before Method B

```routeros
/interface/wireless/access-list/remove [find]
/interface/wireless/connect-list/remove [find]
/interface/wireless/set wlan1 default-authentication=yes
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Access list

**Wireless → Access List → +**: Interface `wlan1`, Signal Strength Range `-75..0`, Authentication and Forwarding ticked. Untick **Default Authenticate** on wlan1.

`TODO` ![Step 1](images/gui/01-access-list.png)

### Step 2 — Block a device

Add a rule with the MAC, Authentication unticked, and move it to the top.

`TODO` ![Step 2](images/gui/02-block.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| No client can connect | `default-authentication=no` and no matching allow rule | Check rule order. [1] |
| Blocked phone still connects | Phone uses a random (private) MAC | Disable MAC randomisation for that network on the phone. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `access-list` | Which clients an AP accepts; first match wins [1] |
| `connect-list` | Which APs a station joins [1] |
| `default-authentication` | Fallback when nothing matches [1] |
| MAC filters | Easy to copy, not security [2] |

## Key Takeaways

- Use signal ranges to keep edge clients from slowing the AP.
- Rules are checked in order; put specific blocks first.

## References

1. [RouterOS Manual — Wireless Interface](https://manual.mikrotik.com/docs/wireless/abgn/wireless-interface/)
2. [RouterOS Manual — HotSpot (Captive portal)](https://manual.mikrotik.com/docs/authentication-authorization-accounting/hotspot-captive-portal/)

---

**Previous:** [F.02 — Wireless Security Profiles](../02-wireless-security-profiles/README.md) · **Next:** F.04 — Virtual AP Guest Network

[Back to the main learning journal](../../README.md)
