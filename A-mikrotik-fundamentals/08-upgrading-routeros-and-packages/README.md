<div align="center">

# Upgrading RouterOS and Packages

*Chapter A — MikroTik Fundamentals | Topic A.08*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

RouterOS is updated regularly, and its features come in packages. This topic checks the installed version and packages, upgrades RouterOS from the internet, and upgrades the RouterBOOT firmware afterwards.

## Why This Matters

Upgrading RouterOS is the first item in MikroTik's hardening guide. [5] The examples in this guide follow the current manual, so an old version can also explain why a command or menu looks different. On a small device like the hAP lite, it also matters which packages are installed, because each one uses flash storage.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.03 — Lab Internet Access](../03-lab-internet-access/README.md) | The router downloads the update itself. |
| [A.07 — Backup, Export, and Restore](../07-backup-export-and-restore/README.md) | Take a backup before upgrading. |

## Theory

### Release channels

RouterOS updates come from four channels: long-term, stable, testing, and development. Each channel offers only its latest version, so switching to a channel with an older release is also how you downgrade. [1]

```routeros
/system/package/update/set channel=stable
```

[1]

### Packages

RouterOS software is distributed as `.npk` packages. `routeros` is the main package; extra packages add features such as `container`, `wifi-qcom`, `user-manager`, or `iot`. [2] Packages can be enabled, disabled, or uninstalled, and the change is applied with `/system/package/apply-changes`. [2]

The `wireless` package conflicts with `wifi-qcom` and `wifi-qcom-ac`. [2] The hAP lite uses `wireless`: its product page lists an SMIPS CPU, 16 MB of flash, and `wireless` as the default wireless package. [3]

### Upgrade paths

| Method | How [1] |
| --- | --- |
| Online | `/system/package/update/check-for-updates`, then `/system/package/update/install`. |
| Download only | `/system/package/update/download`, then reboot later. |
| Manual | Upload `routeros-*.npk` (and matching extras) to the root of Files, then `/system/reboot`. |

Uploaded package files are removed after the reboot. [1][2] The correct package architecture is shown by `/system/resource/print` as `architecture-name`. [2]

### RouterBOOT firmware

On RouterBOARD devices, the bootloader (RouterBOOT) is upgraded separately after RouterOS, with `/system/routerboard/upgrade` followed by a reboot. [1]

### Version numbers

RouterOS version numbers are not decimals: the manual notes that the dot does not mean a decimal point. [1] So 7.10 is newer than 7.9.

### How it connects

```text
A.07 backup ──► A.08 upgrade ──► every later topic runs on a current RouterOS
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version before | `TODO` |
| RouterOS version after | `TODO` |
| Starting state | A.06/A.07 configuration with internet access |

> [!WARNING]
> Do not power off the router during an upgrade. Make a backup (A.07) first.
