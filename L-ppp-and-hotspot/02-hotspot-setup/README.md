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
