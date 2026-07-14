<div align="center">

# WireGuard Road Warrior

*Chapter M — VPN & Network Tunneling | Topic M.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A "road warrior" is a single device, such as a laptop or phone, that connects to the home or office network from anywhere. This topic sets up the MikroTik as a WireGuard server for phones and laptops, following the manual's example, and exports a client configuration.

## Why This Matters

This is the safe way to manage the router and reach home devices from outside: the management ports stay closed (H.02), and only one encrypted UDP port is open.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [M.02 — WireGuard Site-to-Site](../02-wireguard-site-to-site/README.md) | Interfaces, peers, keys. |
| [E.05 — IP Cloud and DDNS](../../E-dhcp-and-network-services/05-ip-cloud-and-ddns/README.md) | A name for the endpoint. |

## Theory

### The manual's road-warrior setup

```routeros
/interface/wireguard add listen-port=13231 name=wireguard1
/ip/address add address=192.168.100.1/24 interface=wireguard1
/interface/wireguard/peers add allowed-address=192.168.100.2/32 interface=wireguard1 public-key="<paste public key from remote device here>"
/ip/firewall/filter add chain=input action=accept comment="allow WireGuard" dst-port=13231 protocol=udp place-before=([find chain=input]->0)
/interface/list/member add interface=wireguard1 list=LAN
```

[1] Each client gets one `/32` in `allowed-address`. Adding `wireguard1` to the `LAN` list lets clients manage the router and reach the LAN under the H.02 rules. [1]

### Client configuration

`/interface/wireguard/peers/show-client-config` shows a configuration for a peer that a client app can import. [1]

### Reachability

The router must be reachable from the internet on UDP 13231. Behind a home router, that needs a port forward there; behind CGN (B.06), inbound connections are not possible, and the site-to-site direction (the router connecting out) must be used instead.

## Lab Environment

| Item | Value |
| --- | --- |
| Server | MikroTik hAP lite, `TODO` version, H.02 firewall |
| Client | Phone with the official WireGuard app, on mobile data |
| Endpoint | DDNS name (E.05) or public address |

## Method A — CLI

### Step 1 — Interface and address

```routeros
/interface/wireguard add listen-port=13231 name=wireguard1
/ip/address add address=192.168.100.1/24 interface=wireguard1
```

[1]

`TODO` ![Step 1](images/cli/01-interface.png)

### Step 2 — Client keys and peer

Create a tunnel in the phone app; it generates its own key pair. Copy the phone's public key into:

```routeros
/interface/wireguard/peers add allowed-address=192.168.100.2/32 interface=wireguard1 public-key="<phone-public-key>"
```

[1]

`TODO` ![Step 2](images/cli/02-peer.png)

### Step 3 — Firewall and LAN membership

```routeros
/ip/firewall/filter add chain=input action=accept comment="allow WireGuard" dst-port=13231 protocol=udp place-before=([find chain=input]->0)
/interface/list/member add interface=wireguard1 list=LAN
```

[1] Add a port forward for UDP 13231 on the home router if the hAP lite is behind it.

`TODO` ![Step 3](images/cli/03-firewall.png)

### Step 4 — Phone settings

In the app: address `192.168.100.2/32`, DNS `192.168.100.1`; peer public key = router's public key; endpoint = DDNS name `:13231`; allowed IPs `192.168.88.0/24, 192.168.100.0/24`.

> [!CAUTION]
> **Privacy:** cover the DDNS name, keys, and public addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 4](images/cli/04-phone.png)

### Step 5 — Test

On mobile data, connect and open WinBox Mobile or ping `192.168.88.1`. On the router:

```routeros
/interface/wireguard/peers/print detail
```

`TODO` ![Step 5](images/cli/05-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Handshake | Recent | `TODO` |
| Phone reaches `192.168.88.1` | Yes | `TODO` |
| WinBox port from internet | Still closed | `TODO` |

## Reset Before Method B

```routeros
/interface/list/member/remove [find interface=wireguard1]
/ip/firewall/filter/remove [find comment="allow WireGuard"]
/interface/wireguard/peers/remove [find]
/ip/address/remove [find interface=wireguard1]
/interface/wireguard/remove wireguard1
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Interface, address, peer

**WireGuard → +**, **IP → Addresses → +**, **WireGuard → Peers → +** as in Method A.

`TODO` ![Step 1](images/gui/01-setup.png)

### Step 2 — Client config

Select the peer and use **Show Client Config** (if available in your WinBox version) to display a configuration the app can import. [1]

`TODO` ![Step 2](images/gui/02-client-config.png)

### Step 3 — Firewall and list

As in Method A Step 3.

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| No handshake from mobile data | UDP 13231 not reachable (home router, CGN) | Port forward upstream; see B.06. |
| Connected but cannot reach LAN | Phone's allowed IPs too narrow, or wireguard1 not in `LAN` | Fix both. [1] |
| Names do not resolve | DNS not set in the app | DNS `192.168.100.1`. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| One `/32` per client in `allowed-address` | Road-warrior peers [1] |
| `wireguard1` in `LAN` | Clients treated as LAN [1] |
| `show-client-config` | Export for the app [1] |

## Key Takeaways

- A road-warrior VPN replaces open management ports.
- Inbound VPN needs a reachable public address.

## References

1. [RouterOS Manual — WireGuard](https://manual.mikrotik.com/docs/virtual-private-networks/wireguard/)

---

**Previous:** [M.02 — WireGuard Site-to-Site](../02-wireguard-site-to-site/README.md) · **Next:** M.04 — L2TP over IPsec

[Back to the main learning journal](../../README.md)
