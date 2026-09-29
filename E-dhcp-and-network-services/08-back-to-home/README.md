<div align="center">

# Back To Home

*Chapter E — DHCP & Network Services | Topic E.08*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

> [!IMPORTANT]
> Back To Home requires an ARM, ARM64, or TILE CPU and RouterOS 7.12 or later. [1] The hAP lite (SMIPS) and CHR on x86 do not qualify, so this topic needs a different MikroTik device. Check `architecture-name` in `/system/resource/print` first.

## Overview

Back To Home turns a MikroTik router into a WireGuard VPN server that phones and computers can reach even when the router has no public address, through a MikroTik relay. This topic explains how it works, what it changes on the router, and how to give and revoke access.

## Why This Matters

M.03 needed a reachable public address and a port forward. Many homes are behind CGN (B.06), where that is impossible. Back To Home solves exactly that case without manual WireGuard configuration.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [E.05 — IP Cloud and DDNS](../05-ip-cloud-and-ddns/README.md) | MikroTik cloud services. |
| [M.03 — WireGuard Road Warrior](../../M-vpn-and-network-tunneling/03-wireguard-road-warrior/README.md) | The manual alternative. |

## Theory

### How it works

The router becomes a WireGuard server. Without a public IP, or behind NAT, a MikroTik relay connects the client and the router. [1] The manual states the connection is end-to-end encrypted and the relay cannot read the traffic. [1] Relayed connections can be slower. [1]

### What it is not

Traffic leaves to the internet from the home router, so websites see the home address. The manual describes it as simple home access, not an anonymity service. [1]

### What enabling it changes

Enabling Back To Home adds a `back-to-home-vpn` WireGuard interface, the addresses `192.168.216.1/24` and `fc00:0:0:216::1/64`, and firewall rules. [1] Revoking removes all of it and deletes all users. [1]

### Users

Users are created under `/ip/cloud/back-to-home-user`. `allow-lan=no` is the default, so a user reaches the internet through home but not the home LAN unless allowed. [1] An `expires` date can be set when the user is created and cannot be changed later. [1] Shares can also be made from the phone app. [1]

### How it connects

```text
E.08 Back To Home (behind CGN) ──► M.03 manual WireGuard (with a public address)
```

## Lab Environment

| Item | Value |
| --- | --- |
| Router | `TODO` an ARM/ARM64 MikroTik device, RouterOS ≥ 7.12 [1] |
| Client | Laptop with the WireGuard app, on a different network (phone hotspot) |

## Method A — CLI

### Step 1 — Check support

```routeros
/system/resource/print
```

`architecture-name` must be ARM, ARM64, or TILE. [1]

`TODO` ![Step 1](images/cli/01-architecture.png)

### Step 2 — Enable

```routeros
/ip/cloud/set back-to-home-vpn=enabled
/ip/cloud/print
```

[1] Wait for `vpn-status: running`.

> [!CAUTION]
> **Privacy:** `/ip/cloud/print` shows the DDNS name (serial number) and public address. Cover them. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-enable.png)

### Step 3 — Create a user and export its config

```routeros
/ip/cloud/back-to-home-user/add name=laptop allow-lan=yes
/ip/cloud/back-to-home-user/show-client-config [find name=laptop]
```

[1] Import the configuration into the laptop's WireGuard app.

`TODO` ![Step 3](images/cli/03-user.png)

### Step 4 — Test from outside

Connect the laptop to a phone hotspot, activate the tunnel, and open the router's LAN address.

`TODO` ![Step 4](images/cli/04-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `vpn-status` | `running` [1] | `TODO` |
| Laptop on hotspot | Reaches the home LAN (`allow-lan=yes`) | `TODO` |
