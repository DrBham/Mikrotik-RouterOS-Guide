<div align="center">

# CAPsMAN (Legacy Wireless)

*Chapter F — Wireless Networking | Topic F.05*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

With several access points, configuring each one by hand quickly becomes error-prone. CAPsMAN (Controlled Access Point system Manager) lets one MikroTik manage the radios of many others, called CAPs. This topic explains legacy CAPsMAN, which is the version for the hAP lite's `wireless` package.

## Why This Matters

Offices, schools, and homes with more than one AP need the same SSID and password everywhere, and changes in one place. CAPsMAN provides that without buying a separate controller.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [F.01 — Wireless AP Basics](../01-wireless-ap-basics/README.md) | Radio settings. |
| [F.02 — Wireless Security Profiles](../02-wireless-security-profiles/README.md) | WPA2. |

## Theory

### Legacy and WiFi CAPsMAN

There are two CAPsMAN systems. Legacy CAPsMAN (`/caps-man`) manages devices running the `wireless` package; WiFi CAPsMAN manages devices with the `wifi-qcom` packages. A WiFi CAP can only join a WiFi CAPsMAN, and a legacy CAP only a legacy CAPsMAN. [1] The hAP lite uses `wireless` (A.03, F.01), so it belongs to legacy CAPsMAN.

### Manager and CAPs

The manager holds the configuration and pushes it to the CAPs, which need only minimal configuration to find the manager. [1][2] The CAP handles the wireless encryption; data is forwarded to the manager, or locally at the CAP when local forwarding is on. [2] Any RouterOS device with at least a level 4 license can be a CAP, and Nv2 and NStreme cannot be used. [2]

### Building blocks

| Menu | Holds [1] |
| --- | --- |
| `/caps-man/manager` | Turn CAPsMAN on (`enabled`, default `no`) |
| `/caps-man/security` | WPA settings |
| `/caps-man/configuration` | SSID, channel, security for radios |
| `/caps-man/provisioning` | Which configuration a CAP radio receives; matched by name, then radio MAC |
| `/caps-man/interface` | Radios currently managed |

If no provisioning rule matches, the implicit action is `create-enabled`. [1]

> [!WARNING]
> **Source needed:** the manual's full setup example sits in a part of the AP Controller page that was only read as a summary. The commands below follow that summary. Verify each against the [AP Controller (CAPsMAN) page](https://manual.mikrotik.com/docs/wireless/abgn/capsman/ap-controller-capsman/) before publishing.

```routeros
/caps-man/security add name="wpa2psk" authentication-types=wpa2-psk encryption=aes-ccm
/caps-man/configuration add name=master-cfg ssid=master security=wpa2psk security.passphrase=12345678 channel.width=20
/caps-man/provisioning add action=create-dynamic-enabled master-configuration=master-cfg
```

## Lab Environment

| Item | Value |
| --- | --- |
| Manager | CHR or any RouterOS device on the LAN, `TODO` version |
| CAP | hAP lite, `TODO` version |
| Network | Both on the same LAN segment |

> [!NOTE]
> A 2.4 GHz configuration is needed for the hAP lite. `TODO` set `channel.band` and frequency for 2.4 GHz after confirming the option names.
