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

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, A.03, NTP synchronised (E.04) |
| Client | Phone with MAC randomisation off for this network |

## Method A — CLI

### Step 1 — Profile

```routeros
/ip/kid-control add name=Lab mon=16h-20h tue=16h-20h wed=16h-20h thu=16h-20h fri=16h-20h sat=9h-21h sun=9h-21h rate-limit=5M
```

[1] Adjust today's hours so the phone is currently outside them, for the test.

`TODO` ![Step 1](images/cli/01-profile.png)

### Step 2 — Device

```routeros
/ip/kid-control/device add name=Phone user=Lab mac-address=00:00:5E:00:53:63
/ip/kid-control/device/print
```

[1] Use the phone's real MAC.

> [!CAUTION]
> **Privacy:** cover real MAC addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-device.png)

### Step 3 — Observe blocking and limits

Outside the hours, the phone has no internet. Change today's hours to include now and test the 5 Mbps limit. Then:

```routeros
/ip/firewall/filter/print where dynamic
/queue/simple/print
/ip/kid-control/device/print stats
```

[1]

`TODO` ![Step 3](images/cli/03-enforced.png)

### Step 4 — Pause and resume

```routeros
/ip/kid-control/pause Lab
/ip/kid-control/resume Lab
```

[1]

`TODO` ![Step 4](images/cli/04-pause.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Outside hours | No internet | `TODO` |
| Inside hours | ~5 Mbps | `TODO` |
| Paused | Blocked until resume [1] | `TODO` |

## Reset Before Method B

```routeros
/ip/kid-control/device/remove [find user=Lab]
/ip/kid-control/remove Lab
```

[1]

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Profile and device

**IP → Kid Control → Kids → +** (hours, rate limit), **Devices → +** (MAC, user).

`TODO` ![Step 1](images/gui/01-kid-control.png)

### Step 2 — Pause

Select the profile and use **Pause** / **Resume**.

`TODO` ![Step 2](images/gui/02-pause.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Device not affected | Random MAC on the phone [1] | Turn off MAC randomisation for this network. |
| Wrong hours | Router clock or time zone [1] | E.04. |
| Limit ignored | FastTrack bypassing queues | Chapter K. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/ip/kid-control add name= mon= … rate-limit=` | Profile [1] |
| `/ip/kid-control/device add user= mac-address=` | Device [1] |
| `pause` / `resume` | Block now / restore [1] |
| IPv4 only, MAC-based | Limits [1] |

## Key Takeaways

- Kid Control is a friendly front end to firewall rules and queues.
- MAC randomisation and a wrong clock are the usual reasons it seems not to work.

## References

1. [RouterOS Manual — Kid Control](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/kid-control/)

---

**Previous:** [K.04 — Queue Tree with Mangle](../04-queue-tree-with-mangle/README.md) · **Next:** [Chapter L — PPP & Hotspot](../../L-ppp-and-hotspot/01-pppoe-server-profiles-and-secrets/README.md)

[Back to the main learning journal](../../README.md)
