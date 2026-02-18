<div align="center">

# Static IP Configuration

*Chapter B — Network Fundamentals & IP Addressing | Topic B.11*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

This topic puts Chapter B into practice. We take ether3 out of the LAN bridge, give it its own static subnet, give the laptop a static address on that subnet, and test which traffic works and why.

## Why This Matters

Routers, servers, printers, and point-to-point links use static addresses so they never change. Doing it by hand once makes every field in B.04 and B.07 concrete: the address, the prefix, the network, and the gateway the client must point to.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses and Subnet Masks](../04-ipv4-addresses-and-subnet-masks/README.md) | Address, mask, network. |
| [B.07 — Gateway and Default Route](../07-gateway-and-default-route/README.md) | Why the client needs a gateway. |
| [A.03 — Lab Internet Access](../../A-mikrotik-fundamentals/03-lab-internet-access/README.md) | Internet access for the last test. |

## Theory

### Addresses in RouterOS

An address is entered as address plus prefix, for example `192.168.88.1/24`, and RouterOS calculates the `network`. [1] Adding an address also creates a connected route to its network. [2] The default configuration puts the LAN address on the bridge, not on the individual ports. [3]

### The plan

| Item | Value | Why |
| --- | --- | --- |
| Router ether3 | `10.10.10.1/24` | A new subnet, outside the LAN `192.168.88.0/24` (B.05) |
| Laptop | `10.10.10.2`, mask `255.255.255.0` | Same network as ether3 (B.04) |
| Laptop gateway | `10.10.10.1` | The router address on this subnet (B.07) |
| Laptop DNS | `10.10.10.1` | The router's resolver (B.08) |

ether3 must leave the LAN bridge first: a port that is a bridge member belongs to the bridge's network. [3]

### Will it reach the internet?

The router's NAT rule from the default configuration translates traffic going out of the WAN interface list. [4] It does not care which LAN subnet the packet came from unless the rule limits `src-address`, so traffic from `10.10.10.0/24` should also be translated. Check your rule in Step 1.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration + A.03 (internet over wlan1) |
| Laptop | Moves from ether2 to **ether3** during the lab |

> [!WARNING]
> Keep the laptop on ether2 until Step 3 is done, so you do not lose access to the router.

## Method A — CLI

### Step 1 — Check the starting point

```routeros
/ip/address/print
/interface/bridge/port/print
/ip/firewall/nat/print
```

`TODO` ![Step 1](images/cli/01-starting-point.png)

### Step 2 — Remove ether3 from the bridge

```routeros
/interface/bridge/port/remove [find interface=ether3]
```

[5][6]

`TODO` ![Step 2](images/cli/02-remove-port.png)

### Step 3 — Add the static address

```routeros
/ip/address/add address=10.10.10.1/24 interface=ether3 comment="B.11 static subnet"
/ip/address/print
```

[1] The `network` column shows `10.10.10.0`.

`TODO` ![Step 3](images/cli/03-add-address.png)

### Step 4 — Configure the laptop

Move the cable to **ether3**. In Windows: **Settings → Network & internet → Ethernet → IP assignment → Edit → Manual**, IPv4 on, address `10.10.10.2`, subnet mask `255.255.255.0`, gateway `10.10.10.1`, DNS `10.10.10.1`. Or in an administrator PowerShell:

```powershell
netsh interface ipv4 set address name="Ethernet" static 10.10.10.2 255.255.255.0 10.10.10.1
netsh interface ipv4 set dnsservers name="Ethernet" static 10.10.10.1 primary
```

`TODO` ![Step 4](images/cli/04-laptop-static.png)

### Step 5 — Test layer by layer

```powershell
ping 10.10.10.1
ping 8.8.8.8
ping google.com
```

| Test | Proves |
| --- | --- |
| `ping 10.10.10.1` | Same subnet, ARP and addressing are right (B.03, B.04) |
| `ping 8.8.8.8` | Gateway, routing, and NAT work (B.07, B.10) |
| `ping google.com` | DNS works (B.08) |

On the router:

```routeros
/ip/route/print where connect
/ip/arp/print where interface=ether3
```

[2][7]

`TODO` ![Step 5](images/cli/05-tests.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Connected route | `10.10.10.0/24` on ether3 [2] | `TODO` |
| ARP entry | Laptop address on ether3 [7] | `TODO` |
| Laptop pings | All three succeed | `TODO` |

## Reset Before Method B

Undo Method A so Method B starts from the same point:

```routeros
/ip/address/remove [find comment="B.11 static subnet"]
/interface/bridge/port/add bridge=bridge interface=ether3
```

Set the laptop back to automatic (DHCP) and reconnect it to ether2.

> [!NOTE]
> `TODO` confirm the default bridge name in your `/interface/bridge/print` output before running the second command.

`TODO` ![Reset](images/gui/00-reset.png)

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Remove ether3 from the bridge

**Bridge → Ports**, select ether3, remove.

`TODO` ![Step 1](images/gui/01-remove-port.png)

### Step 2 — Add the address

**IP → Addresses → +**: Address `10.10.10.1/24`, Interface `ether3`.

`TODO` ![Step 2](images/gui/02-add-address.png)

### Step 3 — Configure the laptop

Same as Method A Step 4, using the Windows settings window.

`TODO` ![Step 3](images/gui/03-laptop-static.png)

### Step 4 — Test

Run the three pings from the laptop, then check **IP → Routes** and **IP → ARP** in WinBox.

`TODO` ![Step 4](images/gui/04-tests.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Cannot ping 10.10.10.1 | ether3 still in the bridge, or wrong laptop mask | Step 2; compare networks (B.04). |
| Can ping 10.10.10.1 but not 8.8.8.8 | Wrong laptop gateway, or NAT limited to `192.168.88.0/24` | Check gateway; check NAT `src-address`. [4] |
| Can ping 8.8.8.8 but not google.com | Laptop DNS not set | Set DNS to `10.10.10.1`. |
| Lost WinBox access | Laptop moved before ether3 was configured | Move back to ether2. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/ip/address/add address=x/y interface=z` | Static address; network calculated [1] |
| Connected route | Created automatically [2] |
| Bridge port | Belongs to the bridge's network [3] |
| Client needs | Address, mask, gateway, DNS |
| Test order | Gateway ping, internet IP, then a name |

## Key Takeaways

- A static client needs the same four values DHCP would have given it.
- Remove a port from the bridge before giving it its own subnet.
- Test by layer: local, routed, then DNS.

## References

1. [RouterOS CLI Reference — ip/address](https://manual.mikrotik.com/docs/cli-reference/ip/address/)
2. [RouterOS Manual — Routing decision](https://manual.mikrotik.com/docs/user-guides/routing-and-networking-protocols/routing-decision/)
3. [RouterOS Manual — Default configurations](https://manual.mikrotik.com/docs/getting-started/configuration-management/default-configurations/)
4. [RouterOS Manual — NAT](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/nat/)
5. [RouterOS CLI Reference — interface/bridge/port](https://manual.mikrotik.com/docs/cli-reference/interface/bridge/port/)
6. [RouterOS Manual — Console](https://manual.mikrotik.com/docs/management-tools/console/)
7. [RouterOS CLI Reference — ip/arp](https://manual.mikrotik.com/docs/cli-reference/ip/arp/)

---

**Previous:** [B.10 — NAT Concepts](../10-nat-concepts/README.md) · **Next:** [Chapter C — Basic Internet Connectivity](../../C-basic-internet-connectivity/01-lab-topology-and-planning/README.md)

[Back to the main learning journal](../../README.md)
