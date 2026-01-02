<div align="center">

# RouterOS CLI Basics

*Chapter A — MikroTik Fundamentals | Topic A.04*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A.02 opened the RouterOS console. This topic teaches how to move around in it, read configuration, change a setting safely, and undo a mistake. The same skills are then shown in WinBox.

After this topic you can:

- Navigate the menu hierarchy and use Tab completion and F1 help.
- Use `print`, `set`, `enable`, `disable`, `find`, and `comment`.
- Protect yourself with safe mode and undo a change.

## Why This Matters

Every later topic gives CLI commands. Reading them is easier once you know how the menu tree works, and you will make fewer mistakes once you know how to target the right item. Safe mode and undo matter as soon as you configure a router over the network: a wrong firewall or address change can cut you off from it.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.02 — Connecting to RouterOS Using WinBox](../02-connecting-to-routeros-using-winbox/README.md) | You need a working CLI and WinBox login. |

## Theory

### The prompt and the menu tree

The prompt shows the user name, the router identity, and the current menu path, for example `[admin@MikroTik] /ip/route>`. [1]

RouterOS commands are organised in a hierarchy of menus. Words in a path can be separated by a space or by `/`, so `/ip route print` and `/ip/route/print` mean the same. [1] You can also enter a menu and run commands inside it:

| Input | Effect [1] |
| --- | --- |
| `/ip/route` | Move into the `/ip/route` menu. |
| `..` | Move up one level. |
| `/` | Move to the top level. |
| `/command` | Run a command from the top level without leaving the current menu. |

### Help and completion

Tab completes a command or argument, and pressing Tab twice lists every possible completion. Abbreviations are accepted when they are unambiguous, so `pin 10.1 c 3 si 100` is read as `ping 10.0.0.1 count 3 size 100`. [1] F1 shows context help for the current menu. [1]

The Up arrow recalls earlier commands, and F3 or Ctrl+R searches the history. Commands that contain passwords are not kept in the history. [1]

### General commands

Most menus share the same commands: `add`, `edit`, `find`, `move`, `print`, `remove`, `set`, `reset`, `export`, `get`, `enable`, `disable`, and `comment`. [1] Once you know them in one menu, you know them everywhere.

### Item numbers, names, and `find`

`print` assigns numbers to the items it lists. These numbers are not constant: they belong to that print output. [1] Several items can be changed at once, either by number or by name:

```routeros
/interface/vlan/set 0,1 mtu=1400
/interface/vlan/set vlan10,vlan20 mtu=1400
```

Names are more stable than numbers, and the manual says not to use item numbers in scripts. Use `find` instead, which selects items by their properties. [1]

### Flags

`print` output marks items with flag letters, explained at the top of the output. For example, `X` means disabled, `R` running, and `I` invalid. [1]

### Safe mode

Ctrl+X (or F4) turns on safe mode, and the prompt shows `<SAFE>`. If the session ends abnormally, for example because a change cut your connection, RouterOS undoes the changes made in safe mode. Pressing Ctrl+X again keeps the changes; Ctrl+D, or `/quit` answered with `y`, undoes them. Safe mode remembers at most 100 changes and does not apply to commands that need a reboot, such as a reset or a backup restore. [1]

### Undo and history

`/undo` and `/redo` step through recent configuration changes, and `/system/history/print` lists them. The history keeps the last 100 changes and is cleared on reboot. [2]

### How it connects

```text
A.02 Log in ──► A.04 Move around and change things safely ──► A.05 Reset ──► A.06 Users and security ──► every later topic
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| WinBox version | `TODO` |
| Starting state | Default configuration, internet access from [A.03](../03-lab-internet-access/README.md) |
