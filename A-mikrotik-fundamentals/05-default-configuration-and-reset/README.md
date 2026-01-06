<div align="center">

# Default Configuration and Reset

*Chapter A — MikroTik Fundamentals | Topic A.05*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A new MikroTik router already contains a configuration written by MikroTik. This topic shows what that configuration contains, how to read it, and the three ways to reset a router: back to the defaults, to an empty configuration, or with the reset button.

## Why This Matters

Most tutorials, including Chapter C of this guide, start from a known state. If the router still has old settings, the same commands give different results. Being able to reset on purpose, and to recover access afterwards, is what makes it safe to experiment in a lab.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.02 — Connecting to RouterOS Using WinBox](../02-connecting-to-routeros-using-winbox/README.md) | MAC login is needed after an empty reset. |
| [A.04 — RouterOS CLI Basics](../04-routeros-cli-basics/README.md) | Reading `print` output. |

## Theory

### Default configuration types

MikroTik ships different default configurations depending on the device: CPE Router, LTE CPE AP router, AP Router, PTP Bridge, W60G Bridge, WISP Bridge, Switch, IP Only, and CAP. [1]

For the AP Router type, ether1 is the WAN port with a DHCP client, the IP firewall, and NAT. The other Ethernet ports and the wireless interfaces are in a LAN bridge with `192.168.88.1/24`, a DHCP server, and DNS. [1] The IP Only type only puts `192.168.88.1/24` on the first port. [1]

The exact commands that build the default configuration can be printed on the router:

```routeros
/system/default-configuration/print
```

[1]

### What a reset does

`/system/reset-configuration` (or **System → Reset Configuration** in WinBox) resets the configuration, including users and the `admin` password, and saves a backup first. [2] RouterOS asks for confirmation before it continues. [3]

| Option | Effect |
| --- | --- |
| *(none)* | Reboot with the default configuration. [2] |
| `no-defaults=yes` | Reboot with no configuration at all. [3] |
| `skip-backup=yes` | Do not save `auto-before-reset.backup` first. [3] |
| `keep-users=yes` | Keep the existing user accounts. [3] |
| `run-after-reset=` | Run your own `.rsc` script after the reset. [3] |

The manual's example for an empty router is:

```routeros
/system/reset-configuration no-defaults=yes skip-backup=yes
```

[3]

### The reset button

Without a login, use the reset button: disconnect power, hold the button, reconnect power, and release when the USR LED starts flashing. The device reboots with the default configuration. Holding the button longer starts other modes (backup RouterBOOT loader, CAPs mode, Netinstall). [2]

### Safe mode does not protect a reset

Safe mode ignores commands that require a reboot, such as resetting the configuration. [4] A reset cannot be undone with Ctrl+X or `/undo`.

### After an empty reset

With `no-defaults=yes` the router has no IP address on any port. You reach it again through a MAC connection in WinBox, which works without an IP address. [5]

### How it connects

```text
A.05 Reset to defaults ──► A.03 internet lab works again
A.05 Reset to empty    ──► Chapter C builds the gateway by hand
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| WinBox version | `TODO` |
| Laptop | On ether2 |

> [!WARNING]
> Both resets erase the A.03 internet setup. Save this page on your phone first. After the topic, reset to defaults and repeat A.03.

## Method A — CLI

### Step 1 — Read the default configuration script

```routeros
/system/default-configuration/print
```

Look for the parts described in the theory: the bridge, the address `192.168.88.1/24`, the DHCP server, the DHCP client on ether1, and the NAT rule. [1]

`TODO` ![Step 1](images/cli/01-default-configuration-print.png)

### Step 2 — Reset to the default configuration

```routeros
/system/reset-configuration
```

Answer `y`. [3] The router reboots. Log in again by SSH at `192.168.88.1` after about a minute.

`TODO` ![Step 2](images/cli/02-reset-defaults.png)

### Step 3 — Find the automatic backup

```routeros
/file/print
```

The reset saved `auto-before-reset.backup`. [3]

`TODO` ![Step 3](images/cli/03-auto-backup-file.png)

### Step 4 — Reset to an empty configuration

```routeros
/system/reset-configuration no-defaults=yes skip-backup=yes
```

[3] After the reboot, SSH to `192.168.88.1` no longer works because the router has no IP address.

`TODO` ![Step 4](images/cli/04-reset-no-defaults.png)

### Step 5 — Reconnect by MAC and check

Open WinBox, connect to the router's MAC address from **Neighbors**, and open **New Terminal**. [5]

```routeros
/ip/address/print
/interface/bridge/print
```

Both lists are empty.

> [!CAUTION]
> **Privacy:** cover the MAC address in the Neighbors list. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 5](images/cli/05-empty-configuration.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| After default reset | `192.168.88.1/24` on the bridge [1] | `TODO` |
| `/file/print` | `auto-before-reset.backup` [3] | `TODO` |
| After empty reset | No addresses, no bridge | `TODO` |
