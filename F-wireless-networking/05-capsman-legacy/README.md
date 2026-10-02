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
