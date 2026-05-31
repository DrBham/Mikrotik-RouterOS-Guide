<div align="center">

# Redirect, Netmap, and NAT Helpers

*Chapter I — Network Address Translation (NAT) | Topic I.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Besides masquerade and dst-nat, RouterOS has NAT actions for special jobs. This topic uses `redirect` to force every LAN device to use the router's DNS, explains `netmap` for 1:1 mapping of a whole subnet, and shows the NAT helpers that fix protocols which carry addresses inside their data.

## Why This Matters

Devices with hard-coded DNS servers ignore DHCP and bypass the router's local names and filtering (E.03). Redirect brings them back. Netmap and helpers appear in office networks with public subnets, VoIP, and FTP.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [I.01 — Port Forwarding with dst-nat](../01-port-forwarding-with-dst-nat/README.md) | dstnat basics. |
| [E.03 — DNS Static Entries and Forwarding](../../E-dhcp-and-network-services/03-dns-static-entries-and-forwarding/README.md) | The local DNS to redirect to. |

## Theory

### Redirect

`action=redirect` is a dstnat that sends the packet to the router itself, on the port given in `to-ports`. [1] The manual's example forces LAN DNS to the router, together with `allow-remote-requests=yes` [1]:

```routeros
/ip/dns set allow-remote-requests=yes
/ip/firewall/nat add chain=dstnat action=redirect protocol=udp dst-port=53 in-interface-list=LAN to-ports=53 comment="DNS to the router"
```

The manual adds the same rule for TCP. [1]

### Netmap

`action=netmap` maps a whole public subnet to a local subnet one to one, in both directions, using a pair of rules. It must be placed before the masquerade rule. [1] The manual warns that it makes every port of the mapped hosts reachable from the internet. [1]

```routeros
/ip/firewall/nat add chain=dstnat action=netmap dst-address=172.16.16.8/29 in-interface-list=WAN to-addresses=10.0.0.8/29 comment="1:1 NAT, to the local addresses"
/ip/firewall/nat add chain=srcnat action=netmap src-address=10.0.0.8/29 out-interface-list=WAN to-addresses=172.16.16.8/29 comment="1:1 NAT, to the public addresses" place-before=[find comment="Masquerade the local network"]
```

[1]

### NAT helpers

Some protocols send addresses and ports inside their data, which plain NAT does not change. RouterOS includes helpers for FTP, TFTP, IRC, H.323, SIP, PPTP, and RTSP under `/ip/firewall/service-port`. [1] The SIP helper has options such as `sip-direct-media` (default on) and a one-hour `sip-timeout`. [1]

### How it connects

```text
I.03 redirect ──► E.03 local DNS for every device ──► Chapter K, L HotSpot also redirects DNS and HTTP
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration + A.03 |
| Laptop | DNS set manually to `8.8.8.8` to simulate a hard-coded device |

## Method A — CLI

### Step 1 — Show the bypass

Set the laptop's DNS to `8.8.8.8` manually. Run `nslookup nas.lan` (the E.03 entry, re-add it if needed): it fails, because the router is bypassed.

`TODO` ![Step 1](images/cli/01-bypass.png)

### Step 2 — Redirect DNS

```routeros
/ip/firewall/nat add chain=dstnat action=redirect protocol=udp dst-port=53 in-interface-list=LAN to-ports=53 comment="DNS to the router"
/ip/firewall/nat add chain=dstnat action=redirect protocol=tcp dst-port=53 in-interface-list=LAN to-ports=53 comment="DNS to the router"
```

[1]

`TODO` ![Step 2](images/cli/02-redirect.png)

### Step 3 — Test

`nslookup nas.lan` now answers, even though the laptop still asks `8.8.8.8`.

`TODO` ![Step 3](images/cli/03-test.png)

### Step 4 — Look at the NAT helpers

```routeros
/ip/firewall/service-port print
```

[1]

`TODO` ![Step 4](images/cli/04-service-ports.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `nas.lan` with DNS `8.8.8.8` | Answered by the router | `TODO` |
| Redirect counters | Increase | `TODO` |
