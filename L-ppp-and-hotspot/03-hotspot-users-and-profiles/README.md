<div align="center">

# HotSpot Users and Profiles

*Chapter L — PPP & Hotspot | Topic L.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A HotSpot becomes useful once users have limits: speed, time, data, and how many devices may share an account. This topic creates a guest profile, a day-pass user, a device that bypasses the login, and a trial login.

## Why This Matters

Voucher Wi-Fi is exactly this: accounts with a time or data limit and a speed profile. The same features let a printer or TV work on a HotSpot network without a browser.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [L.02 — HotSpot Setup](../02-hotspot-setup/README.md) | A working HotSpot. |

## Theory

### User profiles

A user profile holds settings shared by many users. The manual's example [1]:

```routeros
/ip/hotspot/user/profile add name=guest rate-limit=512k/2M session-timeout=1h shared-users=1 address-list=hotspot-guests
```

| Setting | Meaning [1] |
| --- | --- |
| `rate-limit=512k/2M` | Upload, then download |
| `session-timeout=1h` | Session length |
| `shared-users=1` | Only one device at a time per account |
| `address-list=` | Logged-in addresses are added to this firewall list |

### Users and totals

Users can have their own limits, such as total time and total data: [1]

```routeros
/ip/hotspot/user add name=daypass password=ChangeMe-3 limit-uptime=1d limit-bytes-total=2G
```

`/ip/hotspot/user/reset-counters daypass` resets the totals. [1]

### Login methods

The HotSpot profile's `login-by` decides how users log in; the default is `cookie,http-chap`. [1] Other methods include `mac` (log in by MAC address) and `trial`. [1] The manual warns that a MAC address is easy to copy, so MAC login is for devices that cannot show a login page. [1]

### IP bindings

An IP binding with `type=bypassed` lets one device use the network without logging in, for example a printer. [1]

```routeros
/ip/hotspot/ip-binding add mac-address=00:00:5E:00:53:20 type=bypassed comment="office printer"
```

[1]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, L.02 complete |
| Clients | Two phones |

## Method A — CLI

### Step 1 — Guest profile

```routeros
/ip/hotspot/user/profile add name=guest rate-limit=512k/2M session-timeout=1h shared-users=1 address-list=hotspot-guests
```

[1]

`TODO` ![Step 1](images/cli/01-profile.png)

### Step 2 — Users

```routeros
/ip/hotspot/user add name=guest1 password=ChangeMe-2 profile=guest
/ip/hotspot/user add name=daypass password=ChangeMe-3 limit-uptime=1d limit-bytes-total=2G
```

[1]

`TODO` ![Step 2](images/cli/02-users.png)

### Step 3 — Test shared-users

Log in as `guest1` on phone 1, then on phone 2. The second login fails because only one session is allowed. [1]

`TODO` ![Step 3](images/cli/03-shared-users.png)

### Step 4 — Bypass a device

```routeros
/ip/hotspot/ip-binding add mac-address=00:00:5E:00:53:20 type=bypassed comment="lab bypass"
```

[1] Use phone 2's MAC. It now reaches the internet without logging in.

> [!CAUTION]
> **Privacy:** cover real MAC addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 4](images/cli/04-bypass.png)

### Step 5 — Trial login

```routeros
/ip/hotspot/profile set hsprof1 login-by=cookie,http-chap,trial trial-uptime-limit=30m trial-uptime-reset=1d
```

[1] The login page now offers a free 30-minute trial.

`TODO` ![Step 5](images/cli/05-trial.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| guest1 speed | ~512k up / 2M down | `TODO` |
| Second guest1 login | Refused | `TODO` |
| Bypassed phone | No login needed | `TODO` |
| Trial | 30 minutes | `TODO` |

## Reset Before Method B

```routeros
/ip/hotspot/ip-binding/remove [find comment="lab bypass"]
/ip/hotspot/user/remove [find name=guest1]
/ip/hotspot/user/remove [find name=daypass]
/ip/hotspot/user/profile/remove guest
/ip/hotspot/profile set hsprof1 login-by=cookie,http-chap
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — User profile

**IP → Hotspot → User Profiles → +**: Name `guest`, Rate Limit `512k/2M`, Session Timeout `1h`, Shared Users `1`.

`TODO` ![Step 1](images/gui/01-profile.png)

### Step 2 — Users

**IP → Hotspot → Users → +**: the two users with their limits.

`TODO` ![Step 2](images/gui/02-users.png)

### Step 3 — IP binding and trial

**IP → Hotspot → IP Bindings → +** (Type `bypassed`); **Server Profiles → hsprof1 → Login**: tick Trial.

`TODO` ![Step 3](images/gui/03-binding-trial.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| "no more sessions are allowed for user" | `shared-users` reached [1] | Expected; raise it if needed. |
| Rate limit ignored | FastTrack [1] | Chapter K. |
| Phone keeps asking to log in | Randomised MAC changed | Disable MAC randomisation for that network. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| User profile | `rate-limit`, `session-timeout`, `shared-users` [1] |
| `limit-uptime`, `limit-bytes-total` | Per-user totals [1] |
| `login-by` | Login methods incl. `mac`, `trial` [1] |
| IP binding `bypassed` | No login for a device [1] |

## Key Takeaways

- Profiles set the rules, users carry the totals.
- MAC-based access is convenient but easy to spoof.

## References

1. [RouterOS Manual — HotSpot (Captive portal)](https://manual.mikrotik.com/docs/authentication-authorization-accounting/hotspot-captive-portal/)

---

**Previous:** [L.02 — HotSpot Setup](../02-hotspot-setup/README.md) · **Next:** Chapter M — VPN & Network Tunneling

[Back to the main learning journal](../../README.md)
