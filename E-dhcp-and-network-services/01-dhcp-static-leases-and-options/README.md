<div align="center">

# DHCP Static Leases and Options

*Chapter E — DHCP & Network Services | Topic E.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

C.05 gave every client a random address from the pool. This topic gives one device a fixed address through a static lease, limits a client's speed from the lease, and sends extra DHCP options.

## Why This Matters

Printers, servers, cameras, and port-forwarding targets (Chapter I) need addresses that do not change. A static lease keeps the device on DHCP, so it still receives the right gateway and DNS, while its address stays the same. It is safer than typing a static address on the device, which the DHCP server would not know about.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | Leases and options. |
| [C.05 — LAN Bridge, IP, and DHCP Server](../../C-basic-internet-connectivity/05-lan-bridge-ip-and-dhcp-server/README.md) | A running DHCP server. |

## Theory

### Dynamic and static leases

Dynamic leases disappear when they expire or are released; static leases stay. [1] A client with a static lease always receives its address. The static address does not have to be inside the pool, and static addresses are not conflict-checked. [1] Keeping static addresses outside the pool avoids handing the same address to another device.

### Lease time

The default lease time is 30 minutes, and clients renew at 50% and rebind at 87.5% of it. [2][3]

### Options

DHCP options (B.09) can be added beyond gateway and DNS. Where an option is defined decides which one wins: RADIUS first, then the lease, then the server, then the network entry. [1] The server only sends an option the client asks for, unless the option has `force=yes`. [1]

### Per-client rate limit

A static lease can carry a `rate-limit`, which limits that client's speed. [1] (Chapter K covers queues in general.)

### How it connects

```text
E.01 fixed address ──► Chapter I port forwarding target ──► Chapter K per-device limits
```

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | `TODO` |
| Starting state | Default configuration (or C.05) + A.03 internet |
| Client | Laptop on ether2, DHCP |
| Fixed address | `192.168.88.50` |

## Method A — CLI

### Step 1 — Find the laptop's lease

```routeros
/ip/dhcp-server/lease/print
```

[1]

> [!CAUTION]
> **Privacy:** cover MAC addresses and host names. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-leases.png)

### Step 2 — Make it static and change the address

The manual's commands, with the documentation MAC replaced by your laptop's MAC: [1]

```routeros
/ip/dhcp-server/lease/make-static [find mac-address=00:00:5E:00:53:10]
/ip/dhcp-server/lease/set [find mac-address=00:00:5E:00:53:10] address=192.168.88.50
```

`TODO` ![Step 2](images/cli/02-make-static.png)

### Step 3 — Renew on the laptop

```powershell
ipconfig /release
ipconfig /renew
ipconfig
```

The laptop now has `192.168.88.50`.

`TODO` ![Step 3](images/cli/03-renew.png)

### Step 4 — Limit the client's speed

```routeros
/ip/dhcp-server/lease/set [find address=192.168.88.50] rate-limit=10M/10M
```

[1] Run a speed test before and after.

> [!NOTE]
> The default FastTrack rule can bypass queues. If the limit does not apply, see Chapter K. [4]

`TODO` ![Step 4](images/cli/04-rate-limit.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Lease | Static, `192.168.88.50` [1] | `TODO` |
| Laptop address | `192.168.88.50` after renew | `TODO` |
| Speed | About 10 Mbps | `TODO` |

## Reset Before Method B

```routeros
/ip/dhcp-server/lease/remove [find address=192.168.88.50]
```

Renew the laptop's address so it gets a dynamic lease again.

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Make the lease static

**IP → DHCP Server → Leases**, select the laptop, **Make Static**.

`TODO` ![Step 1](images/gui/01-make-static.png)

### Step 2 — Change the address and rate limit

Open the lease: Address `192.168.88.50`, Rate Limit `10M/10M`.

`TODO` ![Step 2](images/gui/02-edit-lease.png)

### Step 3 — Renew and test

Same as Method A Step 3.

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli).

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| Laptop keeps the old address | Old lease not released | `ipconfig /release`, then `/renew`. |
| Two devices with one address | Static address inside the pool given to another device | Use addresses outside the pool; static addresses are not conflict-checked. [1] |
| Rate limit has no effect | FastTrack bypass | See Chapter K. [4] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `lease/make-static` | Keep a lease permanently [1] |
| `lease/set … address=` | Fixed address for that MAC [1] |
| `rate-limit=` on a lease | Per-client speed limit [1] |
| Option precedence | RADIUS, lease, server, network [1] |

## Key Takeaways

- Use static leases, not manual addresses, for devices that need a fixed IP.
- Keep static addresses outside the dynamic pool.

## References

1. [RouterOS Manual — DHCP Server](https://manual.mikrotik.com/docs/network-management/dhcp/server/)
2. [RouterOS Manual — DHCP](https://manual.mikrotik.com/docs/network-management/dhcp/)
3. [RFC 2131 — Dynamic Host Configuration Protocol, §4.4.5](https://www.rfc-editor.org/rfc/rfc2131)
4. [RouterOS Manual — Simple Queue](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/simple-queue/)

---

**Previous:** [D.04 — Port Mirroring with the Switch Chip](../../D-interface-and-bridge/04-port-mirroring-with-the-switch-chip/README.md) · **Next:** E.02 — DHCP Relay

[Back to the main learning journal](../../README.md)
