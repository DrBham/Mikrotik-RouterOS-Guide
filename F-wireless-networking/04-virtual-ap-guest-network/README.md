<div align="center">

# Virtual AP Guest Network

*Chapter F — Wireless Networking | Topic F.04*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

One radio can broadcast more than one network. This topic creates a second SSID for guests on a virtual AP, gives it its own subnet and DHCP server, and stops guests from reaching the main LAN.

## Why This Matters

Guests, smart TVs, and IoT devices should reach the internet without seeing your computers. A separate SSID with its own subnet is the simplest way to do that on a single router, and it previews the segmentation ideas of Chapter J.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [F.02 — Wireless Security Profiles](../02-wireless-security-profiles/README.md) | One profile per network. |
| [C.05 — LAN Bridge, IP, and DHCP Server](../../C-basic-internet-connectivity/05-lan-bridge-ip-and-dhcp-server/README.md) | Building a subnet with DHCP. |
| [B.11 — Static IP Configuration](../../B-network-fundamentals/11-static-ip-configuration/README.md) | A second subnet on the router. |

## Theory

### Virtual AP

A virtual wireless interface is created with `master-interface` set to a physical radio; it shares the radio but has its own SSID and settings. [1] The master must be in `ap-bridge`, `bridge`, `station`, or `wds-slave` mode. [1] The manual's description gives this form:

```routeros
/interface wireless add mode=ap-bridge master-interface=wlan1 ssid=guests security-profile=guests
```

[1] The guest network uses the same channel as the main one, because it is the same radio.

> [!WARNING]
> **Source needed:** this example comes from a summary of the Wireless Interface page. Confirm it on the [Wireless Interface page](https://manual.mikrotik.com/docs/wireless/abgn/wireless-interface/).

### Keeping guests apart

The guest interface gets its own address and DHCP server, outside the LAN bridge (B.11, C.05). It can reach the internet through the same NAT rule if the rule matches it. To stop guests from reaching the LAN, a firewall filter rule in the `forward` chain drops traffic from the guest subnet to the LAN subnet. Filter rules are checked top to bottom and the first match decides. [2]

### How it connects

```text
F.04 separate SSID + subnet ──► Chapter H forward-chain rules ──► Chapter J VLANs for wired guests
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | F.02 complete, ether1 uplink, default firewall and NAT |
| Guest subnet | `192.168.99.1/24` |

## Method A — CLI

### Step 1 — Guest security profile

```routeros
/interface/wireless/security-profiles add name=guests authentication-types=wpa2-psk mode=dynamic-keys wpa2-pre-shared-key="GUEST_PASSWORD"
```

[3]

`TODO` ![Step 1](images/cli/01-guest-profile.png)

### Step 2 — Virtual AP

```routeros
/interface/wireless add name=wlan-guest mode=ap-bridge master-interface=wlan1 ssid=LAB-Guest security-profile=guests disabled=no
```

[1]

`TODO` ![Step 2](images/cli/02-virtual-ap.png)

### Step 3 — Address and DHCP

```routeros
/ip/address/add address=192.168.99.1/24 interface=wlan-guest
/ip/pool/add name=pool-guest ranges=192.168.99.10-192.168.99.254
/ip/dhcp-server/network/add address=192.168.99.0/24 gateway=192.168.99.1 dns-server=192.168.99.1
/ip/dhcp-server/add address-pool=pool-guest interface=wlan-guest name=dhcp-guest
```

[4]

`TODO` ![Step 3](images/cli/03-guest-dhcp.png)

### Step 4 — Block guests from the LAN

```routeros
/ip/firewall/filter add chain=forward action=drop src-address=192.168.99.0/24 dst-address=192.168.88.0/24 comment="guests to LAN" place-before=0
```

[2] Check that the NAT masquerade rule does not limit `src-address` to `192.168.88.0/24` (C.06); if it does, add a second rule for the guest subnet.

`TODO` ![Step 4](images/cli/04-guest-isolation.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Phone on `LAB-Guest` | `192.168.99.x` | `TODO` |
| Guest browses | Works | `TODO` |
| Guest pings `192.168.88.1` or a LAN PC | Fails | `TODO` |

## Reset Before Method B

```routeros
/ip/firewall/filter/remove [find comment="guests to LAN"]
/ip/dhcp-server/remove dhcp-guest
/ip/dhcp-server/network/remove [find address=192.168.99.0/24]
/ip/pool/remove pool-guest
/ip/address/remove [find interface=wlan-guest]
/interface/wireless/remove wlan-guest
/interface/wireless/security-profiles/remove guests
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Virtual AP

**Wireless → + → Virtual**: Master Interface `wlan1`, Mode `ap bridge`, SSID `LAB-Guest`, Security Profile `guests`.

`TODO` ![Step 1](images/gui/01-virtual-ap.png)

### Step 2 — Address and DHCP

**IP → Addresses → +** on `wlan-guest`, then **IP → DHCP Server → DHCP Setup** on `wlan-guest`.

`TODO` ![Step 2](images/gui/02-dhcp.png)

### Step 3 — Firewall rule

**IP → Firewall → Filter Rules → +**: Chain `forward`, Src. Address `192.168.99.0/24`, Dst. Address `192.168.88.0/24`, Action `drop`. Move it to the top.

`TODO` ![Step 3](images/gui/03-filter.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Guest gets no address | DHCP server not on `wlan-guest` | Check the server interface. |
| Guest has no internet | NAT limited to the LAN subnet | Add NAT for `192.168.99.0/24`. |
| Guest can still reach the LAN | Drop rule below an accept rule | Move it up; first match wins. [2] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `master-interface=wlan1` | Virtual AP on the same radio [1] |
| Own subnet + DHCP | Guests separated at layer 3 |
| `forward` drop guest→LAN | Isolation [2] |

## Key Takeaways

- A virtual AP gives a second SSID without a second radio.
- Separation comes from the subnet plus a forward-chain drop rule.

## References

1. [RouterOS Manual — Wireless Interface](https://manual.mikrotik.com/docs/wireless/abgn/wireless-interface/)
2. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)
3. [RouterOS Manual — First Time Configuration](https://manual.mikrotik.com/docs/getting-started/first-time-configuration)
4. [RouterOS Manual — DHCP Server](https://manual.mikrotik.com/docs/network-management/dhcp/server/)

---

**Previous:** [F.03 — Access List and Connect List](../03-access-list-and-connect-list/README.md) · **Next:** Chapter G — Routing Fundamentals

[Back to the main learning journal](../../README.md)
