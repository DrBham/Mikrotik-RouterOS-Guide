<div align="center">

# Wireless AP Basics

*Chapter F — Wireless Networking | Topic F.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A.03 used the hAP lite's radio as a client. This chapter uses it the usual way, as an access point (AP) for phones and laptops. This first topic covers the radio settings: mode, band, channel width, frequency, country, and SSID, and how to see connected clients.

## Why This Matters

Most "slow Wi-Fi" complaints are radio problems: a crowded channel, the wrong width, or a country setting that limits power. These settings come before security and guest networks, because nothing above them works well if the radio is badly configured.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | The legacy `wireless` package and station mode. |

## Theory

### Which driver

The hAP lite is a MIPS device, so it only supports the legacy `wireless` driver under `/interface/wireless`. [1] Features such as WPA3 and fast roaming exist only in the newer `wifi` menu, which this device cannot use. [2]

### Modes

| `mode` | Meaning [3] |
| --- | --- |
| `station` | Client; joins an AP (default) |
| `ap-bridge` | Access point for many clients |
| `bridge` | Access point for a single client |
| `station-pseudobridge`, `station-bridge` | Clients that can be bridged (A.03) |

### Band, width, frequency

| Setting | Notes [3] |
| --- | --- |
| `band` | For 2.4 GHz: `2ghz-b`, `2ghz-b/g`, `2ghz-b/g/n`, `2ghz-onlyg`, `2ghz-onlyn` |
| `channel-width` | `20mhz` by default; 20/40 MHz options double the width |
| `frequency` | Channel centre frequency in MHz; ignored in station modes |
| `country` | Limits bands, frequencies, and transmit power to the local rules; default `etsi` |
| `frequency-mode` | `regulatory-domain` (default) applies the country rules |

MikroTik's overview describes 2.4 GHz as crowded, with three non-overlapping frequencies. [1] On a busy band, a 20 MHz channel is usually the safer choice than 40 MHz.

### SSID

The SSID defaults to the router identity. `hide-ssid=yes` removes it from beacons, but the manual notes this does not improve security. [3]

### Clients

`/interface/wireless/registration-table` lists the connected clients on an AP, with signal strength and rates. [3]

### How it connects

```text
F.01 radio ──► F.02 security ──► F.03 who may connect ──► F.04 guest network
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD), 2.4 GHz 802.11b/g/n [4] |
| RouterOS version | `TODO` |
| Starting state | Default configuration (wlan1 as AP in the LAN bridge) |
| Client | Phone |
| Uplink | ether1 to the home router (wlan1 cannot be AP and A.03 station at once) |

## Method A — CLI

### Step 1 — Read the current settings

```routeros
/interface/wireless/print detail
```

[3]

> [!CAUTION]
> **Privacy:** cover the MAC address and, if personal, the SSID. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-wireless-print.png)

### Step 2 — Survey the air

```routeros
/interface/wireless/scan wlan1 rounds=1
```

[3] Note which frequencies nearby networks use. The scan briefly disconnects clients.

`TODO` ![Step 2](images/cli/02-scan.png)

### Step 3 — Configure the AP

```routeros
/interface/wireless/set wlan1 mode=ap-bridge band=2ghz-b/g/n channel-width=20mhz frequency=2437 country=indonesia ssid=LAB-AP disabled=no
```

[3] Choose the least busy frequency from Step 2 and your own country (press Tab after `country=` for the list).

`TODO` ![Step 3](images/cli/03-ap-settings.png)

### Step 4 — Connect a phone and read the registration table

```routeros
/interface/wireless/registration-table/print
```

[3]

`TODO` ![Step 4](images/cli/04-registration.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Phone sees `LAB-AP` | Yes | `TODO` |
| Registration table | Phone listed with signal | `TODO` |
| Phone gets `192.168.88.x` | Yes (wlan1 is in the LAN bridge) | `TODO` |
