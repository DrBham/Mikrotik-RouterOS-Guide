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
