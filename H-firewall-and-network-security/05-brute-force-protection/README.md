<div align="center">

# Brute-Force Protection

*Chapter H — Firewall & Network Security | Topic H.05*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

If SSH must be reachable from the WAN, password-guessing bots will find it. This topic implements MikroTik's staged address-list rules, which put a source on a one-day blacklist after too many new SSH connections in a short time.

## Why This Matters

The best protection is not exposing SSH at all (A.06, M.03). When it must be exposed, rate-based blocking turns thousands of guesses per day into a few dozen.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [H.04 — Address Lists and RAW Rules](../04-address-lists-and-raw-rules/README.md) | `add-src-to-address-list` and timeouts. |
| [A.06 — Users and Basic Router Security](../../A-mikrotik-fundamentals/06-users-and-basic-router-security/README.md) | Strong passwords and keys. |

## Theory

### The staged lists

Each new SSH connection from the WAN moves its source one stage further [1]:

| Connection | Source added to | Timeout |
| --- | --- | --- |
| 1st | `connection1` | 5 minutes |
| 2nd within 5 min | `connection2` | 15 minutes |
| 3rd within 15 min | `connection3` | 1 hour |
| 4th within 1 hour | `bruteforce_blacklist` (and dropped) | 1 day |

`add-src-to-address-list` does not stop the packet, so the counting rules are written in reverse order, followed by an accept for sources not on the blacklist and a drop. [1] All rules sit before the default "drop all not coming from LAN" rule. [1]

### What it does and does not stop

Each SSH connection allows up to 9 password attempts, so a fast attacker gets about 27 guesses before a one-day block. A source that waits more than 5 minutes between connections never moves past stage 1, which the manual estimates at about 2,600 guesses per day. [1] Key-only login (`/ip/ssh/set password-authentication=no`) closes that gap. [1]

### Exempt yourself

Successful logins also count, so you can block yourself. The manual adds a `secured` address list with an accept rule placed before the counting rules. [1] The rules cover IPv4 only; IPv6 needs the same rules under `/ipv6/firewall/filter`. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Router | hAP lite with default configuration and firewall, `TODO` version |
| Attacker | A device on the upstream (WAN) side |
| SSH | Allowed from the WAN for this lab only |

> [!WARNING]
> Remove the rules and close SSH from the WAN again after the lab.
