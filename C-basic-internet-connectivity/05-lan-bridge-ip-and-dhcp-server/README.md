<div align="center">

# LAN Bridge, IP, and DHCP Server

*Chapter C — Basic Internet Connectivity | Topic C.05*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The router is online (C.02–C.04). Now the LAN side: join ether2–ether4 in a bridge, give the bridge the LAN address, run a DHCP server so clients configure themselves, and let the router answer their DNS queries.

## Why This Matters

This is the half of the default configuration that every client depends on. Without it, a laptop plugged into the router gets `169.254.x.x` (B.06) and cannot even reach the router by IP.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.02 — WAN via DHCP Client](../02-wan-dhcp-client/README.md) | A working WAN and the `LAN` list. |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | Bridges. |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | Pool, options, leases. |

## Theory

### The bridge

To combine several ports into one network, RouterOS creates a bridge interface and adds the ports to it. [1] The bridge learns which MAC address is behind which port (B.03). [1] The LAN address goes on the bridge, as the manual's DHCP example does with `interface=bridge1`. [2]

### The DHCP server's three parts

| Part | Holds | Command menu [2] |
| --- | --- | --- |
| Pool | The range of addresses to lease | `/ip/pool` |
| Network | Gateway and DNS server sent to clients (options 3 and 6) | `/ip/dhcp-server/network` |
| Server | Which interface it answers on, and which pool it uses | `/ip/dhcp-server` |

The `setup` command builds all three at once, but needs the IP address on the interface first. [2] The pool must not contain the router's own address. [2] On a bridge, the server needs at least one real port that receives packets. [2]

### DNS for the LAN

Clients will be told to use `192.168.88.1` as their DNS server. For the router to answer them, `allow-remote-requests=yes` must be set; it makes the router answer on port 53 on all its addresses. [3] That is why C.06/Chapter H must block DNS from the WAN. [3]

### Marking the LAN

The bridge joins the `LAN` interface list. Adding a bridge to a list is not the same as adding its ports. [4]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | C.02 finished (WAN online, `WAN`/`LAN` lists) |
| Laptop | ether2, WinBox by MAC, Ethernet set to automatic (DHCP) |

> [!WARNING]
> Adding ether2 to the bridge can interrupt the MAC WinBox session for a moment. Reconnect from Neighbors if it drops.

## Method A — CLI

### Step 1 — Create the bridge and add ports

```routeros
/interface/bridge/add name=bridge-lan
/interface/bridge/port/add bridge=bridge-lan interface=ether2
/interface/bridge/port/add bridge=bridge-lan interface=ether3
/interface/bridge/port/add bridge=bridge-lan interface=ether4
/interface/bridge/port/print
```

[1][5]

`TODO` ![Step 1](images/cli/01-bridge.png)

### Step 2 — Add the LAN address and list membership

```routeros
/ip/address/add address=192.168.88.1/24 interface=bridge-lan
/interface/list/member/add list=LAN interface=bridge-lan
```

[2][6]

`TODO` ![Step 2](images/cli/02-address-list.png)

### Step 3 — Create the DHCP server

The manual's manual-build commands, adapted to this lab's names and pool: [2]

```routeros
/ip/pool/add name=pool-lan ranges=192.168.88.10-192.168.88.254
/ip/dhcp-server/network/add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1
/ip/dhcp-server/add address-pool=pool-lan interface=bridge-lan name=dhcp-lan
```

Alternatively, run `/ip/dhcp-server/setup` and answer its prompts. [2]

`TODO` ![Step 3](images/cli/03-dhcp-server.png)

### Step 4 — Let the router answer DNS for the LAN

```routeros
/ip/dns/set allow-remote-requests=yes
```

[3]

`TODO` ![Step 4](images/cli/04-dns.png)

### Step 5 — Test from the laptop

```powershell
ipconfig /renew
ipconfig
ping 192.168.88.1
nslookup mikrotik.com 192.168.88.1
```

And on the router:

```routeros
/ip/dhcp-server/lease/print
```

[2] The laptop should have an address from `.10–.254`, gateway and DNS `192.168.88.1`. It cannot reach the internet yet: NAT comes in C.06.

> [!CAUTION]
> **Privacy:** the lease list shows the laptop's MAC address and host name. Cover them. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 5](images/cli/05-laptop-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Laptop IP | `192.168.88.10–254`, gateway `.1` | `TODO` |
| Lease | `bound` in the lease list [2] | `TODO` |
| `nslookup` via 192.168.88.1 | Answer returned [3] | `TODO` |
| `ping 8.8.8.8` from laptop | Fails (no NAT yet) | `TODO` |

## Reset Before Method B

Remove what Method A added, keeping the WAN from C.02:

```routeros
/ip/dhcp-server/remove dhcp-lan
/ip/dhcp-server/network/remove [find address=192.168.88.0/24]
/ip/pool/remove pool-lan
/ip/dns/set allow-remote-requests=no
/interface/list/member/remove [find interface=bridge-lan]
/ip/address/remove [find interface=bridge-lan]
/interface/bridge/remove bridge-lan
```

Removing the bridge also removes its ports. Reconnect by MAC if needed.

`TODO` ![Reset](images/gui/00-reset.png)

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Bridge and ports

**Bridge → Bridge → +**: Name `bridge-lan`. **Bridge → Ports → +** three times: ether2, ether3, ether4.

`TODO` ![Step 1](images/gui/01-bridge.png)

### Step 2 — Address and LAN list

**IP → Addresses → +**: `192.168.88.1/24` on `bridge-lan`. **Interfaces → Interface List → +**: List `LAN`, Interface `bridge-lan`.

`TODO` ![Step 2](images/gui/02-address-list.png)

### Step 3 — DHCP server

**IP → DHCP Server → DHCP Setup**: interface `bridge-lan`, address space `192.168.88.0/24`, gateway `192.168.88.1`, pool `192.168.88.10-192.168.88.254`, DNS `192.168.88.1`. [2]

`TODO` ![Step 3](images/gui/03-dhcp-setup.png)

### Step 4 — DNS

**IP → DNS**: tick **Allow Remote Requests**. [3]

`TODO` ![Step 4](images/gui/04-dns.png)

### Step 5 — Test

Renew the laptop's address and repeat the tests from Method A. Check **IP → DHCP Server → Leases**.

`TODO` ![Step 5](images/gui/05-test.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Laptop gets `169.254.x.x` | DHCP server not on the bridge, or laptop port not in the bridge | Check `/interface/bridge/port/print` and the server interface. [2] |
| DHCP server shows an error on the bridge | No bridge port receiving packets | Make sure the laptop port is a running bridge member. [2] |
| `nslookup` times out | `allow-remote-requests=no` | Step 4. [3] |
| Address conflict reported | Pool includes a static address | Keep the router and static devices outside the pool. [2] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `/interface/bridge/add`, `/port/add` | Join LAN ports [1][5] |
| Address on the bridge | LAN gateway `192.168.88.1/24` [2] |
| Pool, network, server | DHCP server parts [2] |
| `allow-remote-requests=yes` | Router answers LAN DNS [3] |
| `LAN` list member | The bridge, not each port [4] |

## Key Takeaways

- The bridge is the LAN; addresses and the DHCP server belong on it.
- The DHCP network entry is where the client's gateway and DNS come from.
- After this topic the LAN works locally; it reaches the internet only after NAT.

## References

1. [RouterOS Manual — Bridging and Switching](https://manual.mikrotik.com/docs/bridging-and-switching/)
2. [RouterOS Manual — DHCP Server](https://manual.mikrotik.com/docs/network-management/dhcp/server/)
3. [RouterOS Manual — DNS](https://manual.mikrotik.com/docs/network-management/dns/)
4. [RouterOS CLI Reference — interface/list/member](https://manual.mikrotik.com/docs/cli-reference/interface/list/member/)
5. [RouterOS CLI Reference — interface/bridge/port](https://manual.mikrotik.com/docs/cli-reference/interface/bridge/port/)
6. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)

---

**Previous:** [C.04 — WAN via PPPoE Client](../04-wan-pppoe-client/README.md) · **Next:** C.06 — NAT Masquerade

[Back to the main learning journal](../../README.md)
