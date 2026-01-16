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

## Method A — CLI

### Step 1 — Record the current state

```routeros
/system/resource/print
/system/package/print
/system/routerboard/print
```

Note `version`, `architecture-name`, and the free storage, the installed packages, and the current and upgrade firmware versions.

> [!CAUTION]
> **Privacy:** `/system/routerboard/print` shows the serial number. Cover it. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-current-state.png)

### Step 2 — Back up

```routeros
/system/backup/save name=before-upgrade password="YOUR_BACKUP_PASSWORD"
```

[4]

`TODO` ![Step 2](images/cli/02-backup.png)

### Step 3 — Check for updates

```routeros
/system/package/update/set channel=stable
/system/package/update/check-for-updates
```

The status says either that a new version is available or that the system is up to date. [1]

`TODO` ![Step 3](images/cli/03-check-for-updates.png)

### Step 4 — Install

```routeros
/system/package/update/install
```

The router downloads the packages and reboots. [1] Log in again after a few minutes.

`TODO` ![Step 4](images/cli/04-install.png)

### Step 5 — Upgrade RouterBOOT

```routeros
/system/routerboard/upgrade
/system/reboot
```

[1]

`TODO` ![Step 5](images/cli/05-routerboard-upgrade.png)

### Step 6 — Confirm

```routeros
/system/resource/print
/system/routerboard/print
/log/print where message~"install"
```

`TODO` ![Step 6](images/cli/06-confirm.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `version` | Newer than before (or "already up to date") [1] | `TODO` |
| RouterBOOT | current-firmware equals upgrade-firmware | `TODO` |
| Configuration | Users, services, A.03 internet unchanged | `TODO` |

## Reset Before Method B

An upgrade cannot be repeated on the same version. For Method B, either wait for the next release or downgrade first by switching the channel to `long-term` and installing, which the manual describes as the way to move to an older version. [1] Otherwise follow Method B as a check-only walkthrough.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Check the current version

Open **System → Resources** and **System → Packages**.

`TODO` ![Step 1](images/gui/01-resources-packages.png)

### Step 2 — Check for updates

In **System → Packages**, click **Check For Updates**, choose the **stable** channel, and check. The manual describes this button as the way to upgrade from WinBox. [5]

`TODO` ![Step 2](images/gui/02-check-for-updates.png)

### Step 3 — Download and install

Click **Download & Install**. The router reboots.

`TODO` ![Step 3](images/gui/03-download-install.png)

### Step 4 — Upgrade RouterBOOT

Open **System → RouterBOARD**, click **Upgrade**, then **System → Reboot**. [1]

`TODO` ![Step 4](images/gui/04-routerboard-upgrade.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli), using **System → Resources** and **System → RouterBOARD**.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Check for updates fails | No DNS or no internet | `:put [:resolve upgrade.mikrotik.com]` and ping, as the manual suggests. [1] |
| Not enough space | Too many packages on 16 MB flash [3] | Remove unused packages with `/system/package/uninstall`. [2] |
| Package uploaded but nothing happened | Router not rebooted, or wrong architecture | Reboot; check `architecture-name`. [1][2] |
| Wireless disappeared after upgrade | `wireless` package missing | Check `/system/package/print`; install `wireless`. [2][3] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/system/package/update/check-for-updates` | Look for a new version [1] |
| `/system/package/update/install` | Download, install, reboot [1] |
| `channel=stable` / `long-term` | Release channel [1] |
| `/system/routerboard/upgrade` + reboot | Firmware after RouterOS [1] |
| `/system/package/print` | Installed packages [2] |
| `architecture-name` | Which `.npk` files fit [2] |

## Key Takeaways

- Back up, upgrade RouterOS, then upgrade RouterBOOT.
- The hAP lite uses the `wireless` package and has little flash; keep extra packages to a minimum.
- 7.10 is newer than 7.9: RouterOS versions are not decimals.

## References

1. [RouterOS Manual — Upgrade](https://manual.mikrotik.com/docs/getting-started/installation-and-upgrade/upgrade/)
2. [RouterOS Manual — Packages](https://manual.mikrotik.com/docs/getting-started/installation-and-upgrade/packages/)
3. [MikroTik — hAP lite (RB941-2nD)](https://mikrotik.com/product/RB941-2nD)
4. [RouterOS Manual — Backup](https://manual.mikrotik.com/docs/getting-started/configuration-management/backup/)
5. [RouterOS Manual — Securing your router](https://manual.mikrotik.com/docs/getting-started/securing-your-router/)

---

**Previous:** [A.07 — Backup, Export, and Restore](../07-backup-export-and-restore/README.md) · **Next:** [Chapter B — Network Fundamentals](../../B-network-fundamentals/01-what-is-a-computer-network/README.md)

[Back to the main learning journal](../../README.md)
