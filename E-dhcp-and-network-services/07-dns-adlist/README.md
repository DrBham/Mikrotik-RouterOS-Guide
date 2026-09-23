<div align="center">

# DNS Adlist

*Chapter E — DHCP & Network Services | Topic E.07*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The router's DNS resolver can block ad and tracker domains for every device on the LAN, using public block lists. This topic sizes the DNS cache, adds an adlist, tests a blocked name, and allows one name back.

## Why This Matters

Blocking at the router covers phones, TVs, and other devices where installing an ad blocker is not possible. It builds directly on the DNS features from B.08, E.03, and I.03.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [E.03 — DNS Static Entries and Forwarding](../03-dns-static-entries-and-forwarding/README.md) | Static and FWD entries. |
| [I.03 — Redirect, Netmap, and NAT Helpers](../../I-network-address-translation/03-redirect-netmap-and-nat-helpers/README.md) | Forcing devices to use the router's DNS. |

## Theory

### How blocking works

For a name on an adlist, A queries are answered with `0.0.0.0` and AAAA queries with `::`; other record types still come from upstream. [1] Only the exact listed name is blocked, not names below it, and blocked answers have a 2-second TTL. [1]

### Cache size first

Adlist names are stored in the DNS cache, at roughly 75 bytes per name (about 6 MiB for 75,000 names), so the cache must be raised before adding a list. [1] If the cache is too small, the import stops with "adlist read: max cache size reached"; raising the size afterwards does not finish it, so remove and re-add the list. [1] On the 32 MB hAP lite, choose small lists. [2]

### Sources and limits

A list is an HTTPS URL or a file on the router, with one name per line in hosts format or plain. [1] Certificates are checked by default (`ssl-verify=yes`). [1] Up to 64 adlists are supported; names must be lower case and wildcards are not supported. [1] Devices that use another DNS server bypass the router's adlists. [1]

### Allowing a name

There is no allow list. A static FWD entry for the name makes the router resolve it upstream, because FWD entries do not apply adlists. [1]

### Maintenance

Lists are checked for changes every 4 hours; `reload` checks immediately and `pause` stops all adlists for a set time. [1] Disabling a list does not free its memory; only removing it does. [1]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration + A.03 |
| Client | Laptop using `192.168.88.1` for DNS |
| List | A small file-based list first (Step 2), then a URL list if memory allows |
