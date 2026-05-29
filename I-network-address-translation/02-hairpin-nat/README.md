<div align="center">

# Hairpin NAT

*Chapter I — Network Address Translation (NAT) | Topic I.02*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Theory%20%2B%20Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

After I.01, the web server works from outside, but a LAN device that opens the router's public address fails. This topic explains why and fixes it with hairpin NAT, also called NAT loopback.

## Why This Matters

People use the same address or DDNS name (E.05) at home and away. Without hairpin NAT, it works only away from home, which looks like a random fault.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [I.01 — Port Forwarding with dst-nat](../01-port-forwarding-with-dst-nat/README.md) | The forward to fix. |

## Theory

### Why it fails

A LAN client connects to the router's WAN address. The dstnat rule rewrites the destination to the server, which is on the same LAN. The server sees the client's LAN address as the source and replies directly, bypassing the router. The client expected the reply from the WAN address, so it drops it.

### The fix

A `srcnat` masquerade rule for traffic from the LAN to the server makes the router the source, so the server's reply goes back through the router, which reverses both translations. [1] The manual's rule:

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade protocol=tcp src-address=10.0.0.0/24 dst-address=10.0.0.3 out-interface-list=LAN comment="Hairpin NAT"
```

[1]

The dstnat rule must also match connections coming from the LAN: rules limited with `in-interface-list=WAN` do not; rules using `dst-address-type=local` without an interface list, or a fixed `dst-address`, do. [1]

### Alternative: local DNS

A static DNS entry (E.03) that makes the name resolve to the server's LAN address inside the LAN avoids hairpin NAT entirely.

### How it connects

```text
I.01 forward ──► I.02 hairpin ──► E.03 split DNS as the alternative
```

## Lab Environment

| Item | Value |
| --- | --- |
| Starting state | I.01 rule in place, server `192.168.88.50:80` |
| Client | Second LAN device, e.g. a phone on Wi-Fi |

## Method A — CLI

### Step 1 — Reproduce the problem

From the phone on the LAN, open `http://<WAN-address>:8080`: it fails.

`TODO` ![Step 1](images/cli/01-fails.png)

### Step 2 — Let the dstnat rule match LAN traffic

```routeros
/ip/firewall/nat set [find comment="Lab web server"] in-interface-list=""
```

The rule still requires `dst-address-type=local`, so it only matches connections to the router's own addresses. [1]

> [!NOTE]
> `TODO` confirm in the lab that an empty `in-interface-list=""` clears the matcher; otherwise use `!in-interface-list` via WinBox.

`TODO` ![Step 2](images/cli/02-dstnat-any.png)

### Step 3 — Add the hairpin rule

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade protocol=tcp src-address=192.168.88.0/24 dst-address=192.168.88.50 out-interface-list=LAN comment="Hairpin NAT" place-before=0
```

[1]

`TODO` ![Step 3](images/cli/03-hairpin.png)

### Step 4 — Test again

The phone now loads the page. The laptop's web server log shows the router's address as the client.

`TODO` ![Step 4](images/cli/04-works.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| LAN client to WAN address | Works | `TODO` |
| Server log source | `192.168.88.1` (router) | `TODO` |

## Reset Before Method B

```routeros
/ip/firewall/nat/remove [find comment="Hairpin NAT"]
/ip/firewall/nat set [find comment="Lab web server"] in-interface-list=WAN
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Edit the dstnat rule

Clear **In. Interface List** on the "Lab web server" rule.

`TODO` ![Step 1](images/gui/01-dstnat.png)

### Step 2 — Hairpin rule

**IP → Firewall → NAT → +**: Chain `srcnat`, Protocol `tcp`, Src. Address `192.168.88.0/24`, Dst. Address `192.168.88.50`, Out. Interface List `LAN`, Action `masquerade`. Move it to the top.

`TODO` ![Step 2](images/gui/02-hairpin.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Still fails from the LAN | dstnat still limited to WAN | Step 2. [1] |
| Works, but server sees every LAN user as the router | Expected side effect of masquerade | Use split DNS (E.03) if the real source matters. |

## Quick Recap

| Item | Meaning |
| --- | --- |
| Hairpin NAT | srcnat masquerade from LAN to the server [1] |
| dstnat must match LAN traffic | Not only `in-interface-list=WAN` [1] |
| Alternative | Local DNS entry (E.03) |

## Key Takeaways

- A forward that works outside but not inside needs hairpin NAT or split DNS.

## References

1. [RouterOS Manual — NAT](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/nat/)

---

**Previous:** [I.01 — Port Forwarding with dst-nat](../01-port-forwarding-with-dst-nat/README.md) · **Next:** I.03 — Redirect, Netmap, and NAT Helpers

[Back to the main learning journal](../../README.md)
