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

## Reset Before Method B

Use the reset button to return to the default configuration [2], then log in again by IP.

`TODO` ![Reset](images/gui/00-reset-button.png)

## Method B — WinBox (GUI)

### Step 1 — Open Reset Configuration

Open **System → Reset Configuration**. [2]

`TODO` ![Step 1](images/gui/01-reset-configuration-window.png)

### Step 2 — Reset to the defaults

Leave every option unticked and click **Reset Configuration**. Confirm. The router reboots with the default configuration. [2]

`TODO` ![Step 2](images/gui/02-reset-defaults.png)

### Step 3 — Reset to an empty configuration

Open the window again, tick **No Default Configuration** and **Do Not Backup**, and reset. [2][3]

`TODO` ![Step 3](images/gui/03-reset-no-defaults.png)

### Step 4 — Reconnect by MAC

Connect from **Neighbors** using the MAC address, then check **IP → Addresses** is empty. [5]

`TODO` ![Step 4](images/gui/04-empty-configuration.png)

### Step 5 — Return to the defaults

Reset again with every option unticked, or use the reset button, so the lab is ready for A.06. Then repeat A.03 for internet access.

`TODO` ![Step 5](images/gui/05-back-to-defaults.png)

### Verify (WinBox)

| Check | Expected result | Actual result |
| --- | --- | --- |
| After default reset | Address `192.168.88.1/24` in IP → Addresses [1] | `TODO` |
| After empty reset | IP → Addresses is empty | `TODO` |
| After Step 5 | Default configuration back | `TODO` |

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Router not found after empty reset | Looking for an IP that no longer exists | Use the MAC address in Neighbors. [5] |
| Neighbors list empty | Cable on ether1, or MAC session unreliable [5] | Use ether2; retry the MAC connection. |
| Router came up in CAPs or Netinstall mode | Reset button held too long [2] | Repeat, releasing as soon as the USR LED flashes. |
| Internet stopped working | The reset removed the A.03 setup | Repeat A.03. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/system/default-configuration/print` | Show the default script [1] |
| `/system/reset-configuration` | Reset to defaults, after a backup [2][3] |
| `no-defaults=yes` | Empty configuration [3] |
| `skip-backup=yes` | No `auto-before-reset.backup` [3] |
| Reset button | Hold, power on, release when USR flashes [2] |
| After empty reset | Log in by MAC [5] |

## Key Takeaways

- The default configuration already makes a SOHO router a working gateway with ether1 as WAN.
- A reset cannot be undone with safe mode or `/undo`.
- After an empty reset, the MAC connection is the way back in.

## References

1. [RouterOS Manual — Default configurations](https://manual.mikrotik.com/docs/getting-started/configuration-management/default-configurations/)
2. [RouterOS Manual — RouterOS configuration reset](https://manual.mikrotik.com/docs/getting-started/configuration-management/routeros-configuration-reset/)
3. [RouterOS Manual — Configuration Management](https://manual.mikrotik.com/docs/getting-started/configuration-management/)
4. [RouterOS Manual — Console](https://manual.mikrotik.com/docs/management-tools/console/)
5. [RouterOS Manual — MAC server](https://manual.mikrotik.com/docs/management-tools/mac-server/)

---

**Previous:** [A.04 — RouterOS CLI Basics](../04-routeros-cli-basics/README.md) · **Next:** A.06 — Users and Basic Router Security

[Back to the main learning journal](../../README.md)
