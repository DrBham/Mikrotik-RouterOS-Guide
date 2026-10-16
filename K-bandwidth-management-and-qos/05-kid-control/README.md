<div align="center">

# Kid Control

*Chapter K — Bandwidth Management & QoS | Topic K.05*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Kid Control is RouterOS's parental control: a profile sets the hours when a person's devices may use the internet and how fast. This topic creates a profile, assigns a device, and pauses and resumes it.

## Why This Matters

It is a common home request, and it shows in a friendly way how RouterOS combines firewall rules and queues (Chapters H and K) automatically.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [K.02 — Simple Queues](../02-simple-queues/README.md) | Rate limits become queues. |
| [E.04 — NTP, Clock, and Identity](../../E-dhcp-and-network-services/04-ntp-clock-and-identity/README.md) | Schedules depend on the clock. |

## Theory

### Profiles and devices

A profile (`/ip/kid-control`) sets the allowed hours for each day and a `rate-limit`. Devices (`/ip/kid-control/device`) are matched to a profile by MAC address; the router finds their IPv4 address through the ARP table. [1]

| Setting | Meaning [1] |
| --- | --- |
| `mon` … `sun` | Allowed hours that day; `mon=""` blocks the whole day |
| Days not set | Defaults: 7h–21h Mon–Thu, 7h–22h Fri, 6h–22h Sat, 6h–21h Sun |
| `rate-limit` | Per device, each direction, during allowed hours |
| `tur-mon` … `tur-sun` | Hours when the rate limit does not apply (not extra access) |

### How it is enforced

The router adds dynamic firewall rules (`jump to kid-control rules` in the forward chain) and, for rate limits, dynamic simple queues. Blocked devices get reject rules. [1]

### Limitations

Matching is by MAC address, and many phones use random MACs. Time comes from the router clock. Kid Control is IPv4 only. Removing a profile leaves its devices behind, so remove them too. [1]

### The manual's example

```routeros
/ip/kid-control add name=Anna mon=16h-20h tue=16h-20h wed=16h-20h thu=16h-20h fri=16h-20h sat=9h-21h sun=9h-21h rate-limit=5M
/ip/kid-control/device add name=Tablet user=Anna mac-address=00:00:5E:00:53:63
```

[1]
