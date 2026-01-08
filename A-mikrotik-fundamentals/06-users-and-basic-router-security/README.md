<div align="center">

# Users and Basic Router Security

*Chapter A — MikroTik Fundamentals | Topic A.06*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

A router reset to defaults has one account, `admin`, and several management services. This topic creates a personal administrator account, retires `admin`, and switches off the management services and discovery features the lab does not need, following MikroTik's own hardening guide.

## Why This Matters

`admin` is the user name every attacker tries first, and every open management service is one more way in. MikroTik's guide recommends changing the default user name, using strong passwords, keeping management off the internet, and disabling what you do not use. [2] Doing this early means every later lab runs on a router you control.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.04 — RouterOS CLI Basics](../04-routeros-cli-basics/README.md) | `set`, `find`, safe mode. |
| [A.05 — Default Configuration and Reset](../05-default-configuration-and-reset/README.md) | The way back if you lock yourself out. |

## Theory

### Users and groups

The default configuration has one user, `admin`, in the `full` group. Each user belongs to exactly one group, and the group's policies decide what the user may do. [1]

| Default group | Summary [1] |
| --- | --- |
| `read` | Can view, but not change, the configuration. It still includes `reboot` and `sensitive`. |
| `write` | Can change the configuration, but not users and groups (`!policy`). |
| `full` | Everything, including `policy` (managing users). |

The three default groups cannot be deleted. A policy written with `!` in front is revoked. [1]

### Passwords

MikroTik recommends passwords of at least 12 characters, mixing character classes and avoiding dictionary words. Inside quotes, `"`, `\`, and `$` must be escaped with a backslash. [2] `/user/settings` can enforce a minimum length and number of character categories. [1]

### Restricting where a user may log in from

A user's `address` property limits logins to the given network, for example `192.168.88.0/24`. [1]

### Management services

RouterOS runs a set of management services, each on a default port. [4]

| Service | Port [4] |
| --- | --- |
| ftp | 21 |
| ssh | 22 |
| telnet | 23 |
| www (WebFig) | 80 |
| www-ssl | 443 |
| winbox | 8291 |
| api / api-ssl | 8728 / 8729 |

Services cannot be added, only enabled, disabled, or changed. [4] MikroTik's guide disables telnet, ftp, www, and api, and shows how to limit a service to one network with `available-from`. [2] Telnet sends everything unencrypted.

### Layer 2 access and discovery

MAC-Telnet and MAC-WinBox work without an IP address and are not blocked by the IP firewall. [3] In the default configuration they are allowed only on the `LAN` interface list. [3] The guide also shows how to stop Neighbor Discovery from announcing the router. [2]

### Firewall from the WAN

The default firewall already blocks management access from the WAN side, and MikroTik advises against exposing management services to the internet at all; use a VPN such as WireGuard instead. [2]

### How it connects

```text
A.06 own user + fewer services ──► Chapter H firewall builds the next layer of protection
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration with A.03 internet access |

> [!WARNING]
> Do the steps in order. Never disable `admin` before you have logged in successfully with the new account. If you lock yourself out, use the reset button (A.05).
