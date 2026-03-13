<div align="center">

# Ethernet Interfaces and MTU

*Chapter D — Interface & Bridge | Topic D.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Every RouterOS configuration starts at the physical port. This topic reads an Ethernet port's link state, speed, negotiation, error counters, and MTU, and tests the cable, so that layer 1 and 2 problems can be ruled in or out before anything else.

## Why This Matters

A port at the wrong speed, a damaged cable, or a mismatched MTU produce symptoms that look like routing or firewall problems: slow transfers, some websites failing, random drops. C.07 started troubleshooting at layer 3; this topic covers the layers below it.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.03 — Ethernet, MAC Addresses, and ARP](../../B-network-fundamentals/03-ethernet-mac-and-arp/README.md) | Ethernet frames and the 1500-byte limit. |
| [C.07 — Verify and Troubleshoot Connectivity](../../C-basic-internet-connectivity/07-verify-and-troubleshoot-connectivity/README.md) | The layer-by-layer test order. |

## Theory

### Auto-negotiation and speed

`auto-negotiation` is on by default. The manual warns against disabling it on only one end of a link, and gigabit and faster copper links cannot work with it disabled. [1] `advertise` limits which link modes are offered during negotiation; `speed` sets a fixed link mode and only applies when negotiation is off. [1] Since RouterOS 7.12, link modes include duplex, so there is no separate duplex setting. [1]

### Monitoring a port

`/interface/ethernet/monitor` shows the link `status` (`link-ok`, `no-link`, `unknown`), the negotiation result, the `rate`, duplex, and what both ends advertised. [1] The statistics include error counters such as `rx-fcs-error` and `rx-too-long`. [1] A growing FCS error counter points to a bad cable or connector.

### Cable test

`cable-test` checks the copper pairs of a port. [1]

### MTU and L2MTU

| Setting | Layer | Meaning |
| --- | --- | --- |
| `mtu` | 3 | Largest IP packet the interface sends; default 1500. [1][2] |
| `l2mtu` | 2 | Largest frame without the MAC header; room for VLAN tags and other layer-2 headers. [2] |

The maximum supported values appear as read-only `max-l2mtu`. [2] Changing the L2MTU reloads all interfaces, which drops every link for a moment. [2] A mismatch shows up as large packets failing while small ones work; ping with `do-not-fragment` and increasing `size` finds the limit. [3]

### How it connects

```text
D.01 physical port ──► D.02 ports joined in a bridge ──► D.03 loops between bridges (STP)
MTU ──► C.04 PPPoE 1492 ──► Chapter J VLAN tags ──► Chapter M tunnels
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration + A.03 |
| Laptop | ether2 |

## Method A — CLI

### Step 1 — List interfaces and MTU

```routeros
/interface/print
/interface/ethernet/print
```

Note `mtu`, `l2mtu`, and `max-l2mtu` on ether2. [2]

> [!CAUTION]
> **Privacy:** cover MAC addresses. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-interfaces.png)

### Step 2 — Monitor the laptop port

```routeros
/interface/ethernet/monitor ether2
```

[1] Record `status`, `auto-negotiation`, and `rate`. Press Ctrl+C to stop.

`TODO` ![Step 2](images/cli/02-monitor.png)

### Step 3 — Read error counters

```routeros
/interface/ethernet/print stats
```

[1]

`TODO` ![Step 3](images/cli/03-stats.png)

### Step 4 — Test the cable

```routeros
/interface/ethernet/cable-test ether2
```

[1] Repeat on an empty port to see the difference.

`TODO` ![Step 4](images/cli/04-cable-test.png)

### Step 5 — Find the path MTU

```routeros
/ping 8.8.8.8 size=1500 do-not-fragment count=2
/ping 8.8.8.8 size=1400 do-not-fragment count=2
```

[3] Increase or decrease `size` until you find the largest size that passes.

`TODO` ![Step 5](images/cli/05-path-mtu.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| ether2 status | `link-ok`, negotiation `done` [1] | `TODO` |
| ether2 rate | `100Mbps` (hAP lite ports are 10/100) | `TODO` |
| FCS errors | 0 or not increasing | `TODO` |
| Largest DF ping | `TODO` bytes | `TODO` |

> [!NOTE]
> `TODO` confirm the hAP lite port speed from the [product page](https://mikrotik.com/product/RB941-2nD) before publishing.
