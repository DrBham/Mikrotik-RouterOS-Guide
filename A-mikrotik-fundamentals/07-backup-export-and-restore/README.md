<div align="center">

# Backup, Export, and Restore

*Chapter A — MikroTik Fundamentals | Topic A.07*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

RouterOS can save its configuration in two forms: a binary **backup** and a text **export**. This topic makes both, copies them to the laptop, and restores them. It also produces the `configuration.rsc` files that practical topics in this guide publish.

## Why This Matters

`/undo` only covers the last 100 changes and is cleared on reboot. [2] A backup or export taken before a risky change, a RouterOS upgrade, or a reset is what lets you return to a working router. Exports are also how configurations are shared, so it matters which one leaks passwords and which one does not.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.05 — Default Configuration and Reset](../05-default-configuration-and-reset/README.md) | Restoring after a reset. |
| [A.06 — Users and Basic Router Security](../06-users-and-basic-router-security/README.md) | Gives the router a configuration worth saving. |

## Theory

### Backup and export compared

| | Backup | Export |
| --- | --- | --- |
| Format | Binary file, `.backup` [1] | RouterOS commands, `.rsc` text [2] |
| Contents | The whole configuration [1] | Only settings that differ from the defaults, unless `verbose` [2] |
| Encryption | With `password`; `aes-sha256` by default [1] | None; sensitive values are left out unless `show-sensitive` [2] |
| Users | Included | Not included: user accounts and passwords are not exported [2] |
| Where to restore | The same device; it contains the device's MAC addresses [1] | The same router or another router [2] |
| Partial | No | Yes, with `path=` or from inside a menu [2] |

A backup without a password is stored unencrypted. [1] A backup should be restored on the same RouterOS version. [1]

### Export options

| Option | Effect [2] |
| --- | --- |
| `file=name` | Write `name.rsc` instead of printing. |
| `verbose` | Include default values. |
| `terse` | One full command per line. |
| `show-sensitive` | Include passwords and keys. |
| `path=/ip/firewall,!/ip/firewall/nat` | Only these menus; `!` excludes one. |

### Import

`/import name.rsc` runs the commands in the file. It adds to the existing configuration and removes nothing, so an existing entry can make it fail. `dry-run` checks a file without applying it. [2]

### Restore

`/system/backup/load name=… password=…` restores a backup and asks to reboot. The name is given without `.backup`. [1]

### How it connects

```text
A.07 export ──► configuration.rsc in every practical topic
A.07 backup ──► taken before A.08 upgrade and before any risky lab
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | A.06 configuration (own user, services limited) |
