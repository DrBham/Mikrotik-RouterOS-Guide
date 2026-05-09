<div align="center">

# Protecting the LAN

*Chapter H — Firewall & Network Security | Topic H.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

H.02 protected the router. This topic writes the `forward` chain, which protects the devices behind it: it speeds up established traffic with FastTrack, drops invalid packets, allows only useful ICMP, and drops new connections from the WAN unless they were port-forwarded.

## Why This Matters

NAT alone hides the LAN only by accident: it does not decide what is allowed. A proper forward chain makes the policy explicit, and it is what keeps port forwards (Chapter I) from opening more than intended.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.02 — Protecting the Router](../02-protecting-the-router/README.md) | Input chain done; same lab. |

## Theory

### The manual's first forward rules

```routeros
/ip/firewall/filter add chain=forward action=fasttrack-connection connection-state=established,related comment="FastTrack"
/ip/firewall/filter add chain=forward action=accept connection-state=established,related,untracked comment="Accept established, related, untracked"
/ip/firewall/filter add chain=forward action=drop connection-state=invalid log=yes log-prefix=invalid comment="Drop invalid"
/ip/firewall/filter add chain=forward action=jump jump-target=icmp protocol=icmp comment="Jump to ICMP filters"
```

[1] FastTrack must be followed by an accept rule for the same states, because the first packets of a connection are not yet FastTracked. [2]

### An ICMP chain

`jump` sends ICMP to a user-defined chain called `icmp`, where only needed types are accepted: echo reply and request, unreachable messages, fragmentation required, time exceeded, and parameter problem. Everything else is dropped. [1]

```routeros
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=0:0 comment="Echo reply"
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=3:0 comment="Net unreachable"
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=3:1 comment="Host unreachable"
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=3:4 comment="Fragmentation required"
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=8:0 comment="Echo request"
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=11:0 comment="Time exceeded"
/ip/firewall/filter add chain=icmp action=accept protocol=icmp icmp-options=12:0 comment="Parameter problem"
/ip/firewall/filter add chain=icmp action=drop comment="Drop all other types"
```

[1] The type numbers are the ICMP types from B.07: 0 echo reply, 3 unreachable, 8 echo request, 11 time exceeded. [3]

### Dropping unsolicited traffic from the WAN

The last rule drops new connections arriving from the WAN that were not destination-NATed, so only port forwards (Chapter I) can open a connection inward:

```routeros
/ip/firewall/filter add chain=forward action=drop connection-state=new connection-nat-state=!dstnat in-interface-list=WAN comment="Drop new from WAN not DSTNATed"
```

> [!WARNING]
> **Source needed:** this rule's form matches the default configuration's "drop all from WAN not DSTNATed" rule. Confirm it against your router's `/system/default-configuration/print` output (A.05) and cite it from there.

### How it connects

```text
H.03 forward policy ──► Chapter I port forwards pass because they are dstnat ──► Chapter K FastTrack vs queues
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | H.02 complete |
