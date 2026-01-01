<div align="center">

# Lab Internet Access

*Chapter A — MikroTik Fundamentals | Topic A.03*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Option 1 — Wired](#option-1--wired-uplink-on-ether1) · [Option 2 — Wireless](#option-2--wireless-uplink-on-wlan1) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

When a laptop is connected to the MikroTik by cable, it may stop using its own Wi-Fi, so the reader loses access to this guide. This topic gives the lab router an internet connection, so the laptop can browse through the MikroTik while you practise every later topic.

There are two ways to do it, depending on what you have at home:

| Option | Uplink | Configuration needed |
| --- | --- | --- |
| [Option 1 — Wired](#option-1--wired-uplink-on-ether1) | Cable from the home router or ISP modem to **ether1** | None (default configuration) |
| [Option 2 — Wireless](#option-2--wireless-uplink-on-wlan1) | MikroTik **wlan1** connects as a Wi-Fi client to home Wi-Fi or a phone hotspot | Yes |

Most learners do not have a free cable from their ISP modem, so **Option 2 is the main lab method** in this guide.

> [!TIP]
> While you are configuring this topic, the laptop may not have internet yet. Open this page on your phone, or save it before connecting the cable.

## Why This Matters

Every later topic needs a working lab with internet access, so that we can test results such as "the LAN can reach the internet" and so the reader can keep the guide open. This topic is also the first time we change the router's configuration, using ideas that Chapter C explains in depth: WAN and LAN, DHCP client, and NAT.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [A.02 — Connecting to RouterOS Using WinBox](../02-connecting-to-routeros-using-winbox/README.md) | You must be able to log in by CLI and WinBox. |

## Theory

### WAN and LAN on the default configuration

On devices with a default configuration, the first Ethernet port (ether1) is the WAN port. It has a DHCP client, is protected by the IP firewall, and NAT is enabled. The other Ethernet ports and the wireless interfaces are in the LAN bridge, which has the address `192.168.88.1/24`, a DHCP server, and DNS. [2]

In other words, the default configuration already turns the router into an internet gateway, **as long as the internet arrives on ether1**. Option 1 uses exactly that. Option 2 moves the WAN role to the wireless interface.

### Which wireless driver the hAP lite uses

The hAP lite (RB941-2nD) has an SMIPS CPU and a 2.4 GHz 802.11b/g/n radio. [5] MIPS-based devices only support the legacy `wireless` driver, configured in the `/interface/wireless` menu. [3] That is why this topic uses `/interface/wireless`, not the newer `/interface/wifi` shown in some official examples.

### Station mode

The `mode` setting decides what the wireless interface does. `station` is the basic client mode: it finds and connects to an acceptable access point. [4] This is the same thing a phone or laptop does when it joins a Wi-Fi network.

The manual lists separate modes, `station-pseudobridge` and `station-bridge`, for when a station interface must be bridged. [4] We do not want that here: wlan1 becomes the WAN side, so we take it **out** of the LAN bridge and give it its own IP address by DHCP.

### Security profiles

The Wi-Fi password is not set on the interface itself but in a **security profile**. For WPA2 networks, the profile uses `mode=dynamic-keys`, `authentication-types=wpa2-psk`, and the network password in `wpa2-pre-shared-key`. [4][1] The interface then refers to the profile by name.

### DHCP client

A DHCP client asks the upstream network for an IP address. Its status shows progress: `searching...`, `requesting...`, and `bound` when it has a lease. [7] By default it also adds a default route through the received gateway (`add-default-route=yes`) and uses the received DNS servers (`use-peer-dns=yes`). [7][6]

### NAT

The laptop has a private address in `192.168.88.0/24`. The default configuration enables NAT on the WAN side so that LAN devices can reach the internet. [2] The official setup guide does this with a `masquerade` rule in the `srcnat` chain on the WAN interface, or on an interface list named `WAN`. [1] When wlan1 becomes the WAN, the NAT rule must also apply to wlan1.

### How it connects

```text
Option 2 traffic path

Laptop ── ether2 ─[ LAN bridge 192.168.88.1 ]─ hAP lite ─[ wlan1: DHCP client, NAT ]─ ))) Home Wi-Fi / phone hotspot ── Internet
            (DHCP server gives the laptop                     (upstream gives the MikroTik
             192.168.88.x)                                     its own address)
```

Chapter B explains the theory (IP addressing, gateway, DHCP, NAT), and Chapter C builds the same gateway from an empty configuration.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| WinBox version | `TODO` |
| Uplink (Option 2) | `TODO` home Wi-Fi or phone hotspot, 2.4 GHz |
| Laptop | Lenovo LOQ 15IAX9 on **ether2** |
| Starting state | Factory default configuration ([reset steps](../02-connecting-to-routeros-using-winbox/README.md#starting-state-reset-to-defaults)) |

`TODO` ![Lab topology](images/topology.svg)

> [!IMPORTANT]
> The upstream network must **not** use `192.168.88.0/24`, the same subnet as the MikroTik LAN. Check the address your phone or laptop gets from it before you start. Chapter B explains why two networks on a router cannot share a subnet.

> [!IMPORTANT]
> The hAP lite radio is 2.4 GHz only. [5] A phone hotspot set to 5 GHz will not appear in the scan.

---

## Option 1 — Wired uplink on ether1

Use this if you can run a cable from your home router or ISP modem.

1. Reset the router to the default configuration.
2. Connect the home router's LAN port to **ether1**, and the laptop to **ether2**. The official guide uses ether1 for the ISP connection. [1]
3. Wait for the DHCP client on ether1 to reach `bound`:

```routeros
/ip/dhcp-client/print
```

4. Test from the laptop:

```powershell
ping 8.8.8.8
```

No other configuration is needed because the default configuration already has a DHCP client, NAT, and firewall on ether1. [2]

`TODO` ![Option 1 verify](images/cli/00-option1-dhcp-client-bound.png)

---

## Option 2 — Wireless uplink on wlan1

### Method A — CLI

Log in over SSH or a WinBox terminal as in A.02.

#### Step 1 — Look at the default configuration

Before changing anything, see how the default configuration handles NAT and interface lists, so you know what to change.

```routeros
/interface/bridge/port/print
/interface/list/member/print
/ip/firewall/nat/print
```

Note which interface list the masquerade rule uses (for example `out-interface-list=WAN`) and which interfaces are members of that list.

`TODO` ![Step 1](images/cli/01-default-config-check.png)

> [!WARNING]
> **Source needed:** the default-configuration manual page describes NAT and the firewall in words but does not show the interface-list commands. [2] The output of this step is our evidence. `/system/default-configuration/print` shows the exact default script. [2]

#### Step 2 — Scan for the upstream Wi-Fi

```routeros
/interface/wireless/scan wlan1
```

Find your home Wi-Fi or hotspot name (SSID) in the list, then press **Ctrl+C** to stop. [10] Running a scan takes wlan1 off the air while it runs. [4]

`TODO` ![Step 2](images/cli/02-wireless-scan.png)

#### Step 3 — Create a security profile

```routeros
/interface/wireless/security-profiles/add name=uplink mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key="YOUR_WIFI_PASSWORD"
```

| Parameter | Meaning |
| --- | --- |
| `name=uplink` | Profile name, used in Step 5. |
| `mode=dynamic-keys` | WPA/WPA2 mode. [4] |
| `authentication-types=wpa2-psk` | WPA2 with a pre-shared key (password). [4] |
| `wpa2-pre-shared-key` | The upstream Wi-Fi password. [4] |

`TODO` ![Step 3](images/cli/03-security-profile.png) — blur the password before publishing.

#### Step 4 — Remove wlan1 from the LAN bridge

```routeros
/interface/bridge/port/remove [find interface=wlan1]
```

The bridge port menu is `/interface/bridge/port`. [8] `find` selects the entry by its properties instead of by item number. [10] wlan1 is about to become the WAN side, so it must no longer be part of the LAN.

`TODO` ![Step 4](images/cli/04-bridge-port-remove.png)

#### Step 5 — Switch wlan1 to station mode

```routeros
/interface/wireless/set wlan1 mode=station ssid="YOUR_WIFI_NAME" security-profile=uplink disabled=no
```

This replaces the MikroTik's own Wi-Fi access point with a client connection. Then check that it has joined the access point:

```routeros
/interface/wireless/registration-table/print
```

`TODO` ![Step 5](images/cli/05-station-connected.png)

#### Step 6 — Add a DHCP client on wlan1

```routeros
/ip/dhcp-client/add interface=wlan1
/ip/dhcp-client/print
```

Wait until the wlan1 entry shows `bound`. [7] The router now has an address from the upstream network, a default route, and DNS servers. [7]

`TODO` ![Step 6](images/cli/06-dhcp-client-bound.png)

#### Step 7 — Make wlan1 a WAN interface for NAT and the firewall

If Step 1 showed that the masquerade rule uses the `WAN` interface list, add wlan1 to that list:

```routeros
/interface/list/member/add list=WAN interface=wlan1
```

If the masquerade rule uses `out-interface=ether1` instead, add a rule for wlan1, following the official guide's pattern [1][9]:

```routeros
/ip/firewall/nat/add chain=srcnat out-interface=wlan1 action=masquerade
```

`TODO` ![Step 7](images/cli/07-wan-list-member.png)

#### Step 8 — Test from the router

```routeros
/ping 8.8.8.8 count=4
/ping google.com count=4
```

The first test checks routing; the second also checks DNS. [1]

`TODO` ![Step 8](images/cli/08-router-ping.png)

#### Verify (CLI)

From the laptop:

```powershell
ipconfig
ping 8.8.8.8
ping google.com
```

| Check | Expected result | Actual result |
| --- | --- | --- |
| Registration table | One entry for the upstream access point | `TODO` |
| `/ip/dhcp-client/print` | wlan1 status `bound` | `TODO` |
| Router ping 8.8.8.8 / google.com | Replies | `TODO` |
| Laptop IP | `192.168.88.x`, gateway `192.168.88.1` [2] | `TODO` |
| Laptop ping / browser | Replies; this guide opens | `TODO` |

`TODO` ![Verify](images/cli/09-laptop-verify.png)

### Reset Before Method B

Reset the router to the default configuration with the reset button, as in [A.02](../02-connecting-to-routeros-using-winbox/README.md#starting-state-reset-to-defaults). [11] Log in again; the configuration from Method A is gone.

`TODO` ![Reset](images/gui/00-after-reset.png)

### Method B — WinBox (GUI)

> [!NOTE]
> Menu names are written as they appear in WinBox. `TODO` confirm each path against WinBox v4 while taking the screenshots.

#### Step 1 — Look at the default configuration

Open **Bridge → Ports**, **Interfaces → Interface List**, and **IP → Firewall → NAT**. Note the same things as CLI Step 1.

`TODO` ![Step 1](images/gui/01-default-config-check.png)

#### Step 2 — Scan for the upstream Wi-Fi

Open **Wireless**, double-click **wlan1**, and click **Scan**. Start the scan and find your SSID.

`TODO` ![Step 2](images/gui/02-wireless-scan.png)

#### Step 3 — Create a security profile

**Wireless → Security Profiles → +**. Name `uplink`, Mode `dynamic keys`, tick `WPA2 PSK`, enter the WPA2 pre-shared key.

`TODO` ![Step 3](images/gui/03-security-profile.png)

#### Step 4 — Remove wlan1 from the LAN bridge

**Bridge → Ports**, select the `wlan1` entry, click **−** (remove).

`TODO` ![Step 4](images/gui/04-bridge-port-remove.png)

#### Step 5 — Switch wlan1 to station mode

**Wireless → wlan1 → Wireless** tab: Mode `station`, SSID your Wi-Fi name, Security Profile `uplink`. Click **OK** and enable the interface. Check **Wireless → Registration** for the upstream access point.

`TODO` ![Step 5](images/gui/05-station-connected.png)

#### Step 6 — Add a DHCP client on wlan1

**IP → DHCP Client → +**, Interface `wlan1`, **OK**. Wait for status `bound`.

`TODO` ![Step 6](images/gui/06-dhcp-client-bound.png)

#### Step 7 — Make wlan1 a WAN interface

**Interfaces → Interface List → +**, List `WAN`, Interface `wlan1`. (Or add a NAT masquerade rule for wlan1 if the default rule uses ether1, as in CLI Step 7.)

`TODO` ![Step 7](images/gui/07-wan-list-member.png)

#### Step 8 — Test from the router

**Tools → Ping**, address `8.8.8.8`, then `google.com`.

`TODO` ![Step 8](images/gui/08-router-ping.png)

#### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli), using **Wireless → Registration**, **IP → DHCP Client**, and the laptop.

`TODO` ![Verify](images/gui/09-laptop-verify.png)

---

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| SSID not in scan | Hotspot set to 5 GHz; hAP lite is 2.4 GHz only [5] | Set the hotspot to 2.4 GHz. |
| Registration table empty | Wrong SSID or password | Check `ssid` and the security profile. |
| DHCP client stays `searching...` | Not associated with the access point [7] | Fix the wireless connection first. |
| Router can ping, laptop cannot | wlan1 not covered by NAT | Redo Step 7. |
| `ping 8.8.8.8` works, `ping google.com` fails | No DNS | Check `use-peer-dns` on the DHCP client. [7] |
| Upstream gives `192.168.88.x` | Same subnet as the MikroTik LAN | Use another upstream or change the hotspot subnet. Chapter C covers changing the LAN subnet. |
| Lost access after Step 4/5 | Changed the interface you were connected through | Connect by cable on ether2 and log in again. |

## Quick Recap

| Item | Value |
| --- | --- |
| Wired uplink | Cable to ether1, default configuration, nothing to set [2] |
| Wireless driver on hAP lite | Legacy `wireless`, `/interface/wireless` [3][5] |
| Station mode | `mode=station` joins an access point as a client [4] |
| Password | Security profile, `dynamic-keys` + `wpa2-psk` [4] |
| WAN address | `/ip/dhcp-client/add interface=wlan1` [6] |
| NAT | wlan1 in the `WAN` list, or a masquerade rule on wlan1 [1] |
| Checks | Registration table, DHCP `bound`, ping 8.8.8.8 and google.com |

## Key Takeaways

- The default configuration is already an internet gateway when the internet arrives on ether1.
- Moving the WAN to wireless means: take wlan1 out of the LAN, make it a station, give it a DHCP client, and cover it with NAT.
- The configuration file for this topic must never contain a real Wi-Fi password.

## References

1. [RouterOS Manual — First Time Configuration](https://manual.mikrotik.com/docs/getting-started/first-time-configuration)
2. [RouterOS Manual — Default configurations](https://manual.mikrotik.com/docs/getting-started/configuration-management/default-configurations/)
3. [RouterOS Manual — Wireless](https://manual.mikrotik.com/docs/wireless/)
4. [RouterOS Manual — Wireless Interface](https://manual.mikrotik.com/docs/wireless/abgn/wireless-interface/)
5. [MikroTik — hAP lite (RB941-2nD)](https://mikrotik.com/product/RB941-2nD)
6. [RouterOS Manual — DHCP Client](https://manual.mikrotik.com/docs/network-management/dhcp/client/)
7. [RouterOS CLI Reference — ip/dhcp-client](https://manual.mikrotik.com/docs/cli-reference/ip/dhcp-client/)
8. [RouterOS CLI Reference — interface/bridge/port](https://manual.mikrotik.com/docs/cli-reference/interface/bridge/port/)
9. [RouterOS CLI Reference — ip/firewall/nat](https://manual.mikrotik.com/docs/cli-reference/ip/firewall/nat/)
10. [RouterOS Manual — Console](https://manual.mikrotik.com/docs/management-tools/console/)
11. [RouterOS Manual — RouterOS configuration reset](https://manual.mikrotik.com/docs/getting-started/configuration-management/routeros-configuration-reset/)

---

**Previous:** [A.02 — Connecting to RouterOS Using WinBox](../02-connecting-to-routeros-using-winbox/README.md) · **Next:** A.04 — RouterOS CLI Basics (planned)

[Back to the main learning journal](../../README.md)
