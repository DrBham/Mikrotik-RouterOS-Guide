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

The AP Controller page's example, without its second (slave) SSID:

```routeros
/caps-man/security add name="wpa2psk" authentication-types=wpa2-psk encryption=aes-ccm
/caps-man/configuration add name=master-cfg ssid=master security=wpa2psk security.passphrase=12345678 channel.frequency=5180 channel.width=20 channel.band=5ghz-a
/caps-man/provisioning add action=create-dynamic-enabled master-configuration=master-cfg
```

[2] The manual's version also adds `slave-configurations=slave-cfg` to the provisioning rule for a second SSID. [2]

## Lab Environment

| Item | Value |
| --- | --- |
| Manager | CHR or any RouterOS device on the LAN, `TODO` version |
| CAP | hAP lite, `TODO` version |
| Network | Both on the same LAN segment |

> [!NOTE]
> A 2.4 GHz configuration is needed for the hAP lite. Valid `channel.band` values include `2ghz-b/g/n`. [2] If `channel.frequency` is left blank, CAPsMAN picks the least occupied frequency itself. [2]

## Method A — CLI

### Step 1 — Manager

```routeros
/caps-man/manager set enabled=yes
```

[1][2] `enabled` (`yes` or `no`, default `no`) is the manager property that turns CAPsMAN on or off. [1]

`TODO` ![Step 1](images/cli/01-manager.png)

### Step 2 — Security, configuration, provisioning

Enter the three commands from the theory section with a 2.4 GHz band and your own passphrase. The configuration line becomes:

```routeros
/caps-man/configuration add name=master-cfg ssid=master security=wpa2psk security.passphrase=YOUR_STRONG_PASSWORD channel.width=20 channel.band=2ghz-b/g/n
```

[2] `channel.frequency` is left out so CAPsMAN chooses it. [2]

> [!CAUTION]
> **Privacy:** cover the passphrase and SSID if personal. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-config.png)

### Step 3 — Make the hAP lite a CAP

```routeros
/interface/wireless/cap/set enabled=yes interfaces=wlan1 discovery-interfaces=bridge
```

[2][3] CAP settings live in `/interface/wireless/cap`. `enabled` turns the CAP feature on, `interfaces` lists the wireless interfaces the manager controls, and `discovery-interfaces` lists the interfaces used to find the manager. [2][3]

> [!NOTE]
> The manual shows these properties and their print output, but no full `set` line for the manager or the CAP. The Step 1 and Step 3 commands are built from those documented properties and still need a lab check.

`TODO` ![Step 3](images/cli/03-cap.png)

### Step 4 — Check on the manager

```routeros
/caps-man/interface/print
```

[1] The hAP lite's radio appears and is bound (`B` flag).

`TODO` ![Step 4](images/cli/04-interfaces.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| CAP radio on manager | Listed, bound [1] | `TODO` |
| Phone connects to the managed SSID | Yes | `TODO` |

## Reset Before Method B

Disable the CAP on the hAP lite and remove the provisioning, configuration, and security entries on the manager; then `/caps-man/manager set enabled=no`. Reset the hAP lite (A.05) to get its local wireless settings back.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Manager

**CAPsMAN → Manager**: Enabled. Add Security, Configuration, and Provisioning entries.

`TODO` ![Step 1](images/gui/01-manager.png)

### Step 2 — CAP

On the hAP lite: **Wireless → CAP**: Enabled, Interfaces `wlan1`, Discovery Interfaces `bridge`, as in Method A Step 3.

`TODO` ![Step 2](images/gui/02-cap.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| CAP never appears | Manager disabled, or wrong discovery interface | Check both sides. [1] |
| CAP joins but no SSID | No provisioning rule match | Add a rule. [1] |
| A newer AP cannot join | It runs `wifi`, not `wireless` [1] | Use WiFi CAPsMAN for it. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| Legacy vs WiFi CAPsMAN | Separate; match the package [1] |
| `/caps-man/manager` | Controller on/off [1] |
| configuration + provisioning | What each CAP radio gets [1] |
| CAP | Level 4 license or higher [2] |

## Key Takeaways

- Configure once on the manager, apply to every AP.
- The hAP lite can only join legacy CAPsMAN.

## References

1. [RouterOS Manual — CAPsMAN](https://manual.mikrotik.com/docs/wireless/abgn/capsman/)
2. [RouterOS Manual — AP Controller (CAPsMAN)](https://manual.mikrotik.com/docs/wireless/abgn/capsman/ap-controller-capsman/)
3. [MikroTik Help (archived version) — CAPsMAN, CAP configuration properties](https://help.mikrotik.com/docs/x/JgB-/)

---

**Previous:** [F.04 — Virtual AP Guest Network](../04-virtual-ap-guest-network/README.md) · **Next:** [Chapter G — Routing Fundamentals](../../G-routing-fundamentals/01-static-routes/README.md)

[Back to the main learning journal](../../README.md)
