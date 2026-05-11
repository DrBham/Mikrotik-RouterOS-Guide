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

[1] The manual's advanced firewall guide uses the same rule with the comment `defconf: drop all from WAN not DSTNATed`. It protects LAN clients from direct access by someone on the WAN side who knows the LAN range. [4]

> [!NOTE]
> The manual pages do not print the full default configuration. Compare the rule with your router's `/system/default-configuration/print` output (A.05) in the lab.

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

## Method A — CLI

### Step 1 — Forward rules

Turn on safe mode (Ctrl+X), then enter the four forward rules from the theory section. [1]

`TODO` ![Step 1](images/cli/01-forward.png)

### Step 2 — ICMP chain

Enter the eight `icmp` chain rules. [1]

`TODO` ![Step 2](images/cli/02-icmp-chain.png)

### Step 3 — Final WAN drop

Enter the "Drop new from WAN not DSTNATed" rule.

`TODO` ![Step 3](images/cli/03-wan-drop.png)

### Step 4 — Test and read counters

From the laptop: browse, `ping 8.8.8.8`, `tracert 8.8.8.8`. Then:

```routeros
/ip/firewall/filter/print stats where chain=forward or chain=icmp
/ip/firewall/connection/print where fasttrack
```

[1][2]

`TODO` ![Step 4](images/cli/04-stats.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Browsing, ping, tracert | Work | `TODO` |
| FastTrack | Connections with the F flag [2] | `TODO` |
| ICMP counters | Echo and time-exceeded rules count | `TODO` |

## Reset Before Method B

```routeros
/ip/firewall/filter/remove [find chain=forward]
/ip/firewall/filter/remove [find chain=icmp]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Forward and ICMP rules

**IP → Firewall → Filter Rules → +** for each rule. For the jump rule, Action `jump`, Jump Target `icmp`. For ICMP rules, Protocol `icmp`, ICMP Options `type:code`.

`TODO` ![Step 1](images/gui/01-rules.png)

### Step 2 — Counters

Add the Packets and Bytes columns; browse from the laptop and watch them.

`TODO` ![Step 2](images/gui/02-counters.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Some websites hang | ICMP type 3:4 dropped, MTU discovery broken | Keep "Fragmentation required". [1] |
| Traceroute shows only stars | Time exceeded (11:0) dropped | Keep it. [1] |
| Queues stop limiting | FastTrack bypasses them [2] | Chapter K. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| FastTrack + accept established | Fast path, then catch the first packets [1][2] |
| `jump` to `icmp` chain | Allow only needed ICMP types [1] |
| Drop new from WAN not dstnat | Only port forwards open inward |

## Key Takeaways

- Do not drop all ICMP; parts of it keep the network working.
- Port forwards and the forward chain work together through `connection-nat-state`.

## References

1. [RouterOS Manual — Filter](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/)
2. [RouterOS Manual — Connection tracking](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/)
3. [RFC 792 — Internet Control Message Protocol](https://www.rfc-editor.org/rfc/rfc792)
4. [RouterOS Manual — Building Advanced Firewall](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/user-guides/building-advanced-firewall/)

---

**Previous:** [H.02 — Protecting the Router](../02-protecting-the-router/README.md) · **Next:** H.04 — Address Lists and RAW Rules

[Back to the main learning journal](../../README.md)
