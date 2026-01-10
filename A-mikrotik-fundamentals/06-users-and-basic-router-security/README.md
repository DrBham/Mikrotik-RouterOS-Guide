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

## Method A — CLI

### Step 1 — Look at users, groups, and services

```routeros
/user/print
/user/group/print
/ip/service/print
```

`TODO` ![Step 1](images/cli/01-users-groups-services.png)

### Step 2 — Create your own administrator

```routeros
/user/add name=netadmin group=full password="YOUR_STRONG_PASSWORD"
```

The password is mandatory. [1] Use your own name instead of `netadmin` and a password that follows the rules above. [2]

> [!CAUTION]
> **Privacy:** cover the password in the screenshot. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 2](images/cli/02-user-add.png)

### Step 3 — Log in as the new user

Open a second PowerShell window and run `ssh netadmin@192.168.88.1`. Keep the first session open until this works.

`TODO` ![Step 3](images/cli/03-new-user-login.png)

### Step 4 — Disable `admin` and restrict your user

```routeros
/user/disable admin
/user/set netadmin address=192.168.88.0/24
/user/print
```

[2][1] `admin` now shows the `X` flag.

`TODO` ![Step 4](images/cli/04-disable-admin.png)

### Step 5 — Turn off unused services

```routeros
/ip/service/disable telnet,ftp,www,api
/ip/service/set winbox,ssh available-from=192.168.88.0/24
/ip/ssh/set strong-crypto=yes
/ip/service/print
```

[2][4]

`TODO` ![Step 5](images/cli/05-services.png)

### Step 6 — Disable other unused features

```routeros
/tool/bandwidth-server/set enabled=no
/ip/proxy/set enabled=no
/ip/socks/set enabled=no
/ip/upnp/set enabled=no
```

[2] MAC access stays allowed on `LAN` so the recovery path from A.02 keeps working. [3]

`TODO` ![Step 6](images/cli/06-features-off.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `ssh admin@192.168.88.1` | Login refused | `TODO` |
| `ssh netadmin@192.168.88.1` | Login works | `TODO` |
| `/ip/service/print` | telnet, ftp, www, api disabled | `TODO` |
| Internet from laptop | Still works | `TODO` |

## Reset Before Method B

Reset to the default configuration with the reset button (A.05), log in as `admin`, and repeat A.03 if you need internet access while following Method B.

`TODO` ![Reset](images/gui/00-after-reset.png)

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Create your own administrator

**System → Users → +**: Name `netadmin`, Group `full`, password twice. [1]

`TODO` ![Step 1](images/gui/01-user-add.png)

### Step 2 — Log in as the new user

Open a second WinBox window and log in as `netadmin`.

`TODO` ![Step 2](images/gui/02-new-user-login.png)

### Step 3 — Disable `admin` and restrict your user

In **System → Users**, disable `admin`. Open `netadmin` and set **Allowed Address** to `192.168.88.0/24`. [1]

`TODO` ![Step 3](images/gui/03-disable-admin.png)

### Step 4 — Turn off unused services

**IP → Services**: disable telnet, ftp, www, and api. Open winbox and ssh and set **Available From** to `192.168.88.0/24`. [4][2]

`TODO` ![Step 4](images/gui/04-services.png)

### Step 5 — Disable other unused features

**Tools → BTest Server**: disable. **IP → Proxy**, **IP → SOCKS**, **IP → UPnP**: make sure each is disabled. [2]

`TODO` ![Step 5](images/gui/05-features-off.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli), using WinBox logins and **IP → Services**.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Locked out after disabling `admin` | New user's password or address wrong | Reset button (A.05). |
| Login refused from the laptop | Laptop not in the user's `address` network [1] | Check the laptop IP with `ipconfig`. |
| WinBox cannot connect by IP | `available-from` excludes the laptop [2] | Use MAC WinBox from the LAN [3] and fix the setting. |
| Password with `$` rejected or changed | Special characters not escaped [2] | Escape `"`, `\`, `$` with `\`. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/user/add name=… group=full password=…` | New administrator [1] |
| `/user/disable admin` | Retire the default name [2] |
| `/user/set … address=192.168.88.0/24` | Log in only from the LAN [1] |
| `/ip/service/disable telnet,ftp,www,api` | Fewer open services [2] |
| `available-from=` | Limit a service to a network [2] |
| MAC-WinBox on `LAN` only | Default; keeps recovery possible [3] |

## Key Takeaways

- Log in with the new account before disabling `admin`.
- Disable what you do not use and limit the rest to the LAN.
- Management from the internet belongs behind a VPN, not on an open port.

## References

1. [RouterOS Manual — User](https://manual.mikrotik.com/docs/authentication-authorization-accounting/user/)
2. [RouterOS Manual — Securing your router](https://manual.mikrotik.com/docs/getting-started/securing-your-router/)
3. [RouterOS Manual — MAC server](https://manual.mikrotik.com/docs/management-tools/mac-server/)
4. [RouterOS Manual — Services](https://manual.mikrotik.com/docs/system-information-and-utilities/services/)

---

**Previous:** [A.05 — Default Configuration and Reset](../05-default-configuration-and-reset/README.md) · **Next:** A.07 — Backup, Export, and Restore

[Back to the main learning journal](../../README.md)
