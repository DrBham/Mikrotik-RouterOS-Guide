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

## Method A — CLI

### Step 1 — Explore the menu tree

```routeros
/ip
address
print
..
/
```

Then type `/ip/` and press Tab twice to list the submenus, and press F1 inside `/ip/address`. [1]

`TODO` ![Step 1](images/cli/01-menu-tree.png)

### Step 2 — Read the interface list

```routeros
/interface/print
```

Read the flag legend at the top of the output and find `R` (running) on ether2, the port your laptop uses. [1]

> [!CAUTION]
> **Privacy:** use plain `print`. `print detail` shows MAC addresses; if you capture it, cover them. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-interface-print.png)

### Step 3 — Add a comment by name

```routeros
/interface/set ether3 comment="spare LAN port"
/interface/print
```

The comment appears above the item. We used the name `ether3`, not a number. [1]

`TODO` ![Step 3](images/cli/03-comment.png)

### Step 4 — Select an item with `find`

```routeros
/interface/print where comment="spare LAN port"
/interface/disable [find comment="spare LAN port"]
/interface/print
/interface/enable [find comment="spare LAN port"]
```

`find` picks the item by a property, so this works even if the item numbers change. [1] After `disable`, the interface shows the `X` flag. [1]

`TODO` ![Step 4](images/cli/04-find-disable-enable.png)

### Step 5 — Undo a change

```routeros
/system/history/print
/undo
/interface/print
```

`/undo` reverts the most recent change. [2] Run it until the comment from Step 3 is gone.

`TODO` ![Step 5](images/cli/05-undo.png)

### Step 6 — Practise safe mode

1. Press Ctrl+X. The prompt shows `<SAFE>`. [1]
2. Run `/interface/set ether3 comment="safe mode test"`.
3. Press Ctrl+D to leave and undo, then log in again. [1]
4. Run `/interface/print`: the comment is gone.

`TODO` ![Step 6](images/cli/06-safe-mode.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Tab twice after `/ip/` | List of `/ip` submenus [1] | `TODO` |
| ether3 after `disable` | `X` flag [1] | `TODO` |
| After `/undo` | Comment removed [2] | `TODO` |
| After Ctrl+D in safe mode | Change undone [1] | `TODO` |

## Reset Before Method B

Method A undid its own changes, so the router is back where it started. Check with `/interface/print` that ether3 has no comment and is enabled. No reset is needed.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name and button against WinBox v4 while taking the screenshots.

### Step 1 — Find the same menus

Open **Interfaces** and **IP → Addresses**. These windows show the same lists as `/interface/print` and `/ip/address/print`.

`TODO` ![Step 1](images/gui/01-interfaces-window.png)

### Step 2 — Add a comment

Select **ether3** and use the comment button (or open the item and fill in **Comment**). Enter `spare LAN port`.

`TODO` ![Step 2](images/gui/02-comment.png)

### Step 3 — Disable and enable

Select ether3 and use the disable and enable buttons. Watch the item change state in the list.

`TODO` ![Step 3](images/gui/03-disable-enable.png)

### Step 4 — Undo

Use WinBox's undo button, then check that the comment is gone. The change history is the same one `/system/history/print` shows. [2]

`TODO` ![Step 4](images/gui/04-undo.png)

### Step 5 — Safe mode

Click **Safe Mode**, change the comment again, and then disconnect without leaving safe mode. Log in again and check that the change was undone.

`TODO` ![Step 5](images/gui/05-safe-mode.png)

### Verify (WinBox)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Comment shown in Interfaces | `spare LAN port` above ether3 | `TODO` |
| Disabled interface | Shown as disabled | `TODO` |
| Undo | Comment removed | `TODO` |
| Safe mode disconnect | Change undone | `TODO` |

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| `bad command name` | Wrong menu level or typo | Press Tab to complete, or `/` to return to the top level. [1] |
| A command changed the wrong item | Item numbers changed since the last `print` | Use names or `find` instead of numbers. [1] |
| `/undo` does nothing after a reboot | History is cleared on reboot [2] | Use a backup or export (A.07). |
| Safe mode did not undo a reset | Safe mode ignores commands that need a reboot [1] | Use the reset button procedure (A.05). |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/`, `..`, `/ip/route` | Top level, up one level, enter a menu [1] |
| Tab, Tab Tab, F1 | Complete, list completions, help [1] |
| `print`, `set`, `add`, `remove`, `enable`, `disable`, `comment` | General commands in every menu [1] |
| `[find comment="x"]` | Select items by property; safer than numbers [1] |
| Flags `X`, `R`, `I` | Disabled, running, invalid [1] |
| Ctrl+X / F4 | Safe mode [1] |
| `/undo`, `/redo`, `/system/history/print` | Last 100 changes until reboot [2] |

## Key Takeaways

- The same general commands work in every menu, and both space and `/` separate menu levels.
- Item numbers belong to one `print` output; use names or `find` for anything that matters.
- Turn on safe mode before a change that could cut your connection.

## References

1. [RouterOS Manual — Console](https://manual.mikrotik.com/docs/management-tools/console/)
2. [RouterOS Manual — Configuration Management](https://manual.mikrotik.com/docs/getting-started/configuration-management/)

---

**Previous:** [A.03 — Lab Internet Access](../03-lab-internet-access/README.md) · **Next:** A.05 — Default Configuration and Reset

[Back to the main learning journal](../../README.md)
