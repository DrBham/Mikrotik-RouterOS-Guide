<div align="center">

# DNS Static Entries and Forwarding

*Chapter E — DHCP & Network Services | Topic E.03*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

The router's DNS resolver (B.08) can do more than cache answers. This topic adds local names for LAN devices, forwards one domain to a different DNS server, blocks a name, and inspects and flushes the cache.

## Why This Matters

Typing `nas.lan` is easier than remembering `192.168.88.20`, and a small office often needs its internal domain answered by its own server while everything else goes to the internet. These are all features of the same `/ip/dns` menu the LAN already uses.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.08 — DNS Concepts](../../B-network-fundamentals/08-dns-concepts/README.md) | Records, caching, TTL. |
| [E.01 — DHCP Static Leases and Options](../01-dhcp-static-leases-and-options/README.md) | A device with a fixed address to name. |

## Theory

### Static entries

Static entries answer queries from the router's clients without asking any upstream server. [1] An IPv4 address creates an A record and an IPv6 address an AAAA record, and the router also answers the matching reverse (PTR) query. [1] `match-subdomain=yes` makes an entry answer for all names below it as well. [1] Other types include CNAME, MX, TXT, and NXDOMAIN (answer "no such name"). [1]

### Forwarding (FWD)

A `type=FWD` entry sends queries for one domain to a chosen server instead of the normal upstream servers. [1] A named forwarder is a group of servers that the router queries in turn. [1]

### The cache

`/ip/dns/cache/print` shows what the router has cached and `/ip/dns/cache/flush` empties it, which is useful after changing an entry. [1] Static entries have a default TTL of one day. [1]

### Prerequisite: serving the LAN

Clients only use these entries if they use the router as their DNS server and `allow-remote-requests=yes` is set (C.05). [1]

### How it connects

```text
E.03 local names ──► Chapter I hairpin NAT is often avoided with a static DNS entry
                 ──► Chapter M VPN: forward the office domain to the office DNS server
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration + A.03 internet |
| Laptop | ether2, DNS `192.168.88.1` from DHCP |

## Method A — CLI

### Step 1 — Add local names

```routeros
/ip/dns/static/add name=nas.lan address=192.168.88.20
/ip/dns/static/add name=files.lan type=CNAME cname=nas.lan
/ip/dns/static/print
```

[1]

`TODO` ![Step 1](images/cli/01-static.png)

### Step 2 — Test from the laptop

```powershell
nslookup nas.lan 192.168.88.1
nslookup files.lan 192.168.88.1
nslookup 192.168.88.20 192.168.88.1
```

The last line tests the reverse lookup. [1]

`TODO` ![Step 2](images/cli/02-nslookup.png)

### Step 3 — Block a name

```routeros
/ip/dns/static/add name=tracker.example.com type=NXDOMAIN
```

[1] `nslookup tracker.example.com 192.168.88.1` now returns "non-existent domain".

`TODO` ![Step 3](images/cli/03-nxdomain.png)

### Step 4 — Forward one domain

```routeros
/ip/dns/static/add name=example.org type=FWD forward-to=9.9.9.9 match-subdomain=yes
```

[1] Queries for `example.org` and its subdomains now go to `9.9.9.9`.

`TODO` ![Step 4](images/cli/04-fwd.png)

### Step 5 — Inspect and flush the cache

```routeros
/ip/dns/cache/print
/ip/dns/cache/flush
```

[1]

`TODO` ![Step 5](images/cli/05-cache.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| `nas.lan` | `192.168.88.20` | `TODO` |
| `files.lan` | CNAME to `nas.lan` | `TODO` |
| Reverse `192.168.88.20` | `nas.lan` | `TODO` |
| `tracker.example.com` | Non-existent domain | `TODO` |
