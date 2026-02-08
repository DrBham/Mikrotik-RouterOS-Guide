<div align="center">

# DNS Concepts

*Chapter B — Network Fundamentals & IP Addressing | Topic B.08*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

People type names like `mikrotik.com`; IP needs addresses. The Domain Name System (DNS) turns one into the other. This topic covers the parts of DNS, the common record types, caching, and how a MikroTik router acts as a DNS resolver for its LAN.

## Why This Matters

"I can ping 8.8.8.8 but not google.com" is a DNS problem, and you met that exact test in A.03. The default MikroTik configuration also makes the router a DNS server for the LAN, which is useful but dangerous if the WAN side can reach it.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.02 — OSI and TCP/IP Models](../02-osi-and-tcp-ip-models/README.md) | DNS is an application-layer protocol over UDP and TCP. |
| [B.07 — Gateway and Default Route](../07-gateway-and-default-route/README.md) | DNS servers are usually on another network. |

## Theory

### Three parts of DNS

RFC 1034 describes three major components [1]:

| Component | What it is [1] |
| --- | --- |
| Domain name space and resource records | A tree of names, with data attached to each name. |
| Name servers | Programs that hold parts of the tree. A server is authoritative for the parts (zones) it has complete information about. |
| Resolvers | Programs that query name servers on behalf of clients. |

Your laptop runs a resolver; the MikroTik router can be a caching name server for it.

### Names

A domain name is the list of labels from a node up to the root of the tree, for example `www` + `mikrotik` + `com`. Each label is at most 63 octets. [1]

### Resource records

| Type | Value [2] | Holds [1] |
| --- | --- | --- |
| A | 1 | A host address (IPv4) |
| NS | 2 | The authoritative name server for a domain |
| CNAME | 5 | The canonical name of an alias |
| PTR | 12 | A pointer to another part of the name space (used for reverse lookups) |
| MX | 15 | A mail exchange for the domain |

### Caching and TTL

Every record carries a TTL in seconds that says how long a resolver may cache it before discarding it. [1] Caching is why the second lookup of a name is instant, and also why a changed record can take time to be seen everywhere.

### Recursive and non-recursive

A server can answer a query fully on the client's behalf (recursive) or return referrals for the client to follow (non-recursive). Iterative service is required; recursion is optional. [1]

### Transport

DNS messages use server port 53 over UDP and over TCP. [2] A UDP message is limited to 512 bytes; longer answers are truncated, with a flag telling the client to retry. [2]

### How it connects

```text
B.08 DNS ──► B.09 DHCP hands out the DNS server address
         ──► C.03 static WAN needs DNS set by hand; C.02/C.04 receive it automatically
         ──► Chapter H drop DNS requests from the WAN
```
