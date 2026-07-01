<div align="center">

# HotSpot Setup

*Chapter L — PPP & Hotspot | Topic L.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A HotSpot is a captive portal: a device joins the network, but every web page it opens is replaced by a login page until the user signs in. This topic builds one on the guest Wi-Fi with the setup wizard.

## Why This Matters

Cafés, hotels, schools, and voucher-based Wi-Fi all work this way. RouterOS includes it, together with per-user time, traffic, and speed limits (L.03).

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [F.04 — Virtual AP Guest Network](../../F-wireless-networking/04-virtual-ap-guest-network/README.md) | A separate guest interface. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | Guests still get addresses by DHCP. |

## Theory

### What the HotSpot does

A device on a HotSpot interface gets an IP address and can resolve DNS names, but every HTTP web page it opens is replaced by the login page; other traffic is rejected until login. [1] Users can be authenticated from a local list or RADIUS, and limited by time, traffic, and rate. [1]

### Before you start

| Point | From the manual [1] |
| --- | --- |
| IPv4 only | HotSpot works with IPv4. |
| One server per interface | Use a bridge to cover several ports. |
| Management interface | If HotSpot runs on the interface you manage the router from, you lose access until you log in. |
| Rate limits | Create simple queues, which do not limit FastTracked connections. |

### The setup wizard

`/ip/hotspot/setup` asks for the interface, the network address, masquerading, the address pool, a certificate, an SMTP server, DNS servers, a DNS name, and a first user. [1] It creates a HotSpot profile, the server, a user, and a masquerade rule. [1]

### HTTPS

Modern browsers open HTTPS pages, which the HotSpot cannot replace; devices usually detect the portal with their own HTTP check. Logging in over HTTPS needs a certificate valid for the HotSpot's DNS name and trusted by devices. [1]

### Walled garden

The walled garden lists sites that can be opened without logging in, for example a payment page. [1]

```routeros
/ip/hotspot/walled-garden add dst-host=example.com
```

[1]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version |
| HotSpot interface | `wlan-guest` from F.04 (with its address `192.168.99.1/24`) |
| Management | Laptop on ether2 (not on the HotSpot) |
| Client | Phone on `LAB-Guest` |

## Method A — CLI

### Step 1 — Remove the F.04 DHCP server

The wizard creates its own DHCP server and pool. Remove the F.04 ones on `wlan-guest` first (F.04 reset section, DHCP lines only).

### Step 2 — Run the wizard

```routeros
/ip/hotspot/setup
```

[1] Answer: interface `wlan-guest`, local address `192.168.99.1/24`, masquerade yes, pool `192.168.99.10-192.168.99.254`, certificate none, SMTP `0.0.0.0`, DNS `192.168.99.1`, DNS name `hotspot.lan`, user `guest1` with a password.

`TODO` ![Step 2](images/cli/02-wizard.png)

### Step 3 — Inspect what was created

```routeros
/ip/hotspot print
/ip/hotspot/profile print
/ip/hotspot/user print
```

[1]

`TODO` ![Step 3](images/cli/03-created.png)

### Step 4 — Log in from the phone

Join `LAB-Guest`, open any `http://` page (or wait for the phone's login prompt), and log in as `guest1`. Then:

```routeros
/ip/hotspot/host print
/ip/hotspot/active print
```

[1]

> [!CAUTION]
> **Privacy:** cover MAC addresses and device names. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 4](images/cli/04-login.png)

### Step 5 — Walled garden

```routeros
/ip/hotspot/walled-garden add dst-host=example.com
```

[1] Log out and check that `http://example.com` opens without login.

`TODO` ![Step 5](images/cli/05-walled-garden.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Phone before login | Login page | `TODO` |
| After login | Internet works; listed in `/ip/hotspot/active` | `TODO` |
| example.com before login | Opens | `TODO` |
