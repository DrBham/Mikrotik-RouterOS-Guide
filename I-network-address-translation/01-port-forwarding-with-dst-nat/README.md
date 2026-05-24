<div align="center">

# Port Forwarding with dst-nat

*Chapter I — Network Address Translation (NAT) | Topic I.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Source NAT (C.06) lets the LAN out. Destination NAT lets selected connections in: a connection to a port on the router's WAN address is rewritten to a server inside the LAN. This topic publishes a web server running on the laptop.

## Why This Matters

Game servers, CCTV recorders, home NAS boxes, and remote desktops are often reached this way. It is also the most common way people open their network to the internet by mistake, so it is worth doing carefully.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.10 — NAT Concepts](../../B-network-fundamentals/10-nat-concepts/README.md) | dstnat and connection tracking. |
| [E.01 — DHCP Static Leases and Options](../../E-dhcp-and-network-services/01-dhcp-static-leases-and-options/README.md) | The server needs a fixed address. |
| [H.03 — Protecting the LAN](../../H-firewall-and-network-security/03-protecting-the-lan/README.md) | The forward chain lets dstnat traffic in. |

## Theory

### dst-nat

A `dstnat` rule with `action=dst-nat` rewrites the destination address (and optionally port) of incoming connections. [1] The manual's examples [1]:

```routeros
/ip/firewall/nat add chain=dstnat action=dst-nat protocol=tcp dst-address=172.16.16.1 dst-port=22 to-addresses=10.0.0.3 comment="SSH to the server"
/ip/firewall/nat add chain=dstnat action=dst-nat protocol=tcp dst-address=172.16.16.1 dst-port=8080 to-addresses=10.0.0.3 to-ports=80 comment="Web server on port 8080"
```

When the WAN address changes (DHCP, PPPoE), match the WAN instead of a fixed address, using `in-interface-list=WAN` and `dst-address-type=local`. [1]

```routeros
/ip/firewall/nat add chain=dstnat action=dst-nat protocol=tcp dst-port=8443 in-interface-list=WAN dst-address-type=local to-addresses=10.0.0.3 to-ports=443 comment="Web server, any public address"
```

[1]

### Firewall

The default firewall accepts connections that were destination-NATed, so a port forward needs no extra filter rule on a default configuration. [1] H.03's "drop new from WAN not DSTNATed" rule works the same way.

### Risk

The manual warns that a port forward opens the service to everyone on the internet. [1] Restrict it with `src-address` or a source address list (H.04) when you can, and never forward management ports such as WinBox.

### Limits in the A.03 lab

The hAP lite's WAN is behind your home router's NAT, so a test from the real internet will not reach it. Test from the upstream network instead: a device there connects to the hAP lite's WAN address. If your ISP uses CGN (B.06), real port forwarding from the internet is not possible at all.

### How it connects

```text
I.01 dst-nat ──► I.02 hairpin NAT (reach the forward from inside) ──► E.05 DDNS name for the WAN address
```

## Lab Environment

| Item | Value |
| --- | --- |
| Router | MikroTik hAP lite, `TODO` version, default configuration + A.03 or ether1 uplink |
| Server | Laptop `192.168.88.50` (static lease), running a web server on port 80 |
| Tester | A phone or PC on the upstream network |

Start a simple web server on the laptop, for example with Python: `python -m http.server 80` in an empty folder.

## Method A — CLI

### Step 1 — Find the WAN address

```routeros
/ip/address/print where interface=wlan1
```

Use `ether1` if that is your WAN.

`TODO` ![Step 1](images/cli/01-wan-address.png)

### Step 2 — Add the port forward

```routeros
/ip/firewall/nat add chain=dstnat action=dst-nat protocol=tcp dst-port=8080 in-interface-list=WAN dst-address-type=local to-addresses=192.168.88.50 to-ports=80 comment="Lab web server"
```

[1]

`TODO` ![Step 2](images/cli/02-dst-nat.png)

### Step 3 — Test from the upstream network

On the tester, open `http://<hAP-lite-WAN-address>:8080`. The laptop's directory listing appears.

`TODO` ![Step 3](images/cli/03-test.png)

### Step 4 — Restrict the source (optional)

```routeros
/ip/firewall/nat set [find comment="Lab web server"] src-address=<tester-address>
```

[1] Only the tester can connect now.

`TODO` ![Step 4](images/cli/04-restrict.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Tester opens `:8080` | Web page from the laptop | `TODO` |
| Rule counters | Increase | `TODO` |
| Other device (after Step 4) | Blocked | `TODO` |
