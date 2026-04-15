<div align="center">

# Wireless Security Profiles

*Chapter F — Wireless Networking | Topic F.02*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic secures the access point from F.01 with WPA2 and a strong pre-shared key, using a security profile, and explains which options the legacy wireless package offers.

## Why This Matters

An open or weakly protected AP lets anyone join your LAN, which is behind the firewall. The security profile is the one setting that decides who can even start talking to the network.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [F.01 — Wireless AP Basics](../01-wireless-ap-basics/README.md) | A working AP. |

## Theory

### Security profiles

In the legacy package, encryption settings live in a security profile under `/interface/wireless/security-profiles`, and each wireless interface refers to one with `security-profile` (default `default`). [1] One profile can serve several interfaces.

### WPA2 personal

For WPA2 with a password, the profile uses `mode=dynamic-keys`, `authentication-types=wpa2-psk`, and the password in `wpa2-pre-shared-key`. [1][2] The official setup guide's example:

```routeros
/interface/wireless/security-profiles add name="myProfile" authentication-types=wpa2-psk mode=dynamic-keys wpa2-pre-shared-key="YOUR_STRONG_PASSWORD"
```

[2]

### What the legacy package does not have

WPA3 and fast roaming are only available in the newer `wifi` menu. [3] The hAP lite cannot run it (F.01), so WPA2 with a long random key is the best available option on this device.

> [!WARNING]
> **Source needed:** cipher options (`aes-ccm`, `tkip`) and management frame protection settings come from a summary of the Wireless Interface page, not verified text. Confirm on the [Wireless Interface page](https://manual.mikrotik.com/docs/wireless/abgn/wireless-interface/) before relying on them.

### How it connects

```text
F.02 who has the key ──► F.03 which devices are allowed ──► F.04 separate key for guests
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | F.01 Method A complete |

## Method A — CLI

### Step 1 — Look at the profiles

```routeros
/interface/wireless/security-profiles/print
```

[1]

`TODO` ![Step 1](images/cli/01-profiles.png)

### Step 2 — Create a WPA2 profile

```routeros
/interface/wireless/security-profiles add name=lab-wpa2 authentication-types=wpa2-psk mode=dynamic-keys wpa2-pre-shared-key="YOUR_STRONG_PASSWORD"
```

[2] Use at least 12 random characters, as for router passwords (A.06).

> [!CAUTION]
> **Privacy:** cover the key. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-profile-add.png)

### Step 3 — Apply it to wlan1

```routeros
/interface/wireless/set wlan1 security-profile=lab-wpa2
```

[1] The phone disconnects and must enter the new key.

`TODO` ![Step 3](images/cli/03-apply.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Phone shows WPA2 | Yes | `TODO` |
| Wrong key | Rejected | `TODO` |
| Registration table | Phone listed after correct key | `TODO` |
