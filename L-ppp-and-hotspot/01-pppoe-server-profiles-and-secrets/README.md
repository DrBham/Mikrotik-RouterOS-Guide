<div align="center">

# PPPoE Server, Profiles, and Secrets

*Chapter L — PPP & Hotspot | Topic L.01*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

C.04 was the customer side of PPPoE. This topic is the provider side: a PPPoE server with user accounts (secrets), profiles that assign addresses and speed limits, and the list of active sessions. It is the basis of many small ISPs and RT/RW networks.

## Why This Matters

Per-user login, per-user speed, and the ability to disconnect a user are what a small network provider needs. RouterOS provides all three through PPP profiles and secrets.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [C.04 — WAN via PPPoE Client](../../C-basic-internet-connectivity/04-wan-pppoe-client/README.md) | PPPoE, MTU 1492. |
| [K.01 — Queue Concepts and HTB](../../K-bandwidth-management-and-qos/01-queue-concepts-and-htb/README.md) | Rate limits create queues. |

## Theory

### Profiles and secrets

| Object | Holds [1] |
| --- | --- |
| Profile (`/ppp/profile`) | Settings shared by many users: `local-address`, `remote-address`, `rate-limit`, `only-one`, `dns-server`, `address-list` |
| Secret (`/ppp/secret`) | One user: name, password, `service`, `profile`, optional fixed `remote-address` |
| Active (`/ppp/active`) | Users connected now |

`local-address` and `remote-address` in a profile can be an address or the name of an IP pool; a secret's address overrides the profile. [1] Settings are combined in the order: secret, then the secret's profile, then the server's default profile. [1]

### Rate limit and sessions

`rate-limit=5M/10M` is upload/download, and the router adds a dynamic simple queue for each connected user. [1] `only-one=yes` refuses a second session of the same user across all PPP servers. [1]

### The PPPoE server

The server runs on the client-facing interface, which must not have an IP address or a DHCP server, or devices could use the network without logging in. [2] The built-in PPP profiles set `change-tcp-mss=yes`, which avoids the 1492-byte MTU problem from C.04. [2]

```routeros
/ip/pool/add name=pppoe-pool ranges=10.0.0.10-10.0.0.100
/ppp/profile/add name=subscribers local-address=10.0.0.1 remote-address=pppoe-pool
/ppp/secret/add name=alice password=Example-pass-1 profile=subscribers service=pppoe
/interface/pppoe-server/server/add service-name=internet interface=ether3 default-profile=subscribers disabled=no
```

[2]

## Lab Environment

| Item | Value |
| --- | --- |
| Server | MikroTik hAP lite, `TODO` version, internet via ether1 or A.03 |
| Client port | ether3, removed from the LAN bridge, no IP |
| Client | Windows laptop (PPPoE dial-up) or a second MikroTik |

## Method A — CLI

### Step 1 — Free ether3

```routeros
/interface/bridge/port/remove [find interface=ether3]
```

### Step 2 — Pool, profile, secret, server

Enter the four commands from the theory section. [2]

`TODO` ![Step 2](images/cli/02-server.png)

### Step 3 — Speed and single session

```routeros
/ppp/profile/set subscribers rate-limit=5M/10M only-one=yes
```

[1]

`TODO` ![Step 3](images/cli/03-profile.png)

### Step 4 — NAT for subscribers

```routeros
/ip/firewall/nat add chain=srcnat action=masquerade src-address=10.0.0.0/24 out-interface-list=WAN comment="PPPoE subscribers"
```

[3]

### Step 5 — Connect a client

On Windows: **Settings → Network & internet → Dial-up → Set up a new connection → Connect to the Internet → Broadband (PPPoE)**, user `alice`. Then on the router:

```routeros
/ppp/active/print
/queue/simple/print
```

[1] The session and its dynamic queue appear.

> [!CAUTION]
> **Privacy:** cover MAC addresses in `/ppp/active`. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 5](images/cli/05-active.png)

### Step 6 — Disconnect the user

```routeros
/ppp/active/remove [find name=alice]
```

[1]

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Client address | From `10.0.0.10–100` | `TODO` |
| Speed | ~5/10 Mbps | `TODO` |
| Second login as alice | Refused (`only-one`) [1] | `TODO` |
