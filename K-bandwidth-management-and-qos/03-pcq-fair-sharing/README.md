<div align="center">

# PCQ Fair Sharing

*Chapter K — Bandwidth Management & QoS | Topic K.03*

![Level](https://img.shields.io/badge/Level-Intermediate-2563EB) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

Writing one simple queue per device does not scale. Per Connection Queue (PCQ) gives every device its own limit with a single queue. This topic sets a 10 Mbps download and 5 Mbps upload per device for the whole LAN.

## Why This Matters

Cafés, boarding houses, and offices want "everyone gets the same share" without maintaining a list of addresses. PCQ does exactly that, and new devices are covered automatically.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [K.02 — Simple Queues](../02-simple-queues/README.md) | Simple queues and FastTrack. |

## Theory

### Queue types

A queue's `queue` setting chooses its algorithm, or queue type. Types include FIFO (`pfifo`, `bfifo`), `sfq`, `pcq`, `codel`, `fq-codel`, and `cake`. [1] A simple queue has two types (upload and download); a queue tree has one. [1]

### PCQ

PCQ gives every device on a network its own speed limit with one queue. [2] It splits traffic into sub-streams by a classifier and limits each sub-stream to `pcq-rate`. [2]

| Direction | `pcq-classifier` | Why |
| --- | --- | --- |
| Download | `dst-address` | Downloads are addressed to the LAN device [2] |
| Upload | `src-address` | Uploads come from the LAN device [2] |

`pcq-limit` is the queue size in KiB, not a speed. [2] RouterOS also ships default types `pcq-upload-default` and `pcq-download-default`. [1]

### The manual's example

```routeros
/queue/type add name=PCQ_download kind=pcq pcq-rate=10M pcq-classifier=dst-address
/queue/type add name=PCQ_upload kind=pcq pcq-rate=5M pcq-classifier=src-address
/queue/simple add name=lan-pcq target=192.168.88.0/24 queue=PCQ_upload/PCQ_download
```

[2] FastTrack must be disabled first, as in K.02. [2]

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration + A.03 |
| Clients | Laptop and a phone |

## Method A — CLI

### Step 1 — Disable FastTrack

```routeros
/ip/firewall/filter disable [find action=fasttrack-connection]
```

[2]

### Step 2 — Queue types

```routeros
/queue/type add name=PCQ_download kind=pcq pcq-rate=10M pcq-classifier=dst-address
/queue/type add name=PCQ_upload kind=pcq pcq-rate=5M pcq-classifier=src-address
```

[2]

`TODO` ![Step 2](images/cli/02-types.png)

### Step 3 — One queue for the LAN

```routeros
/queue/simple add name=lan-pcq target=192.168.88.0/24 queue=PCQ_upload/PCQ_download
```

[2]

`TODO` ![Step 3](images/cli/03-queue.png)

### Step 4 — Test with two devices

Run speed tests on both devices at the same time, then on one alone. Each is capped at about 10/5 Mbps in both cases.

```routeros
/queue/simple print stats
```

[2]

`TODO` ![Step 4](images/cli/04-test.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| One device | ~10 down / 5 up | `TODO` |
| Two devices together | Each ~10 / 5 | `TODO` |

## Reset Before Method B

```routeros
/queue/simple remove lan-pcq
/queue/type remove [find name~"PCQ_"]
/ip/firewall/filter enable [find action=fasttrack-connection]
```

## Method B — WinBox (GUI)

> [!NOTE]
> `TODO` confirm each menu name against WinBox v4 while taking the screenshots.

### Step 1 — Queue types

**Queues → Queue Types → +**: Type Name `PCQ_download`, Kind `pcq`, Rate `10M`, Classifier `Dst. Address`. Repeat for upload with `Src. Address`, 5M.

`TODO` ![Step 1](images/gui/01-types.png)

### Step 2 — Simple queue

**Queues → Simple Queues → +**: Target `192.168.88.0/24`; Advanced tab: Queue Type upload `PCQ_upload`, download `PCQ_download`.

`TODO` ![Step 2](images/gui/02-queue.png)

### Verify (WinBox)

Same checks as [Verify (CLI)](#verify-cli). Re-enable FastTrack when finished.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
| --- | --- | --- |
| All devices share 10 Mbps total | Classifier wrong (e.g. none) | Use `dst-address` / `src-address`. [2] |
| No limit | FastTrack on | Disable it. [2] |
| Upload and download swapped | Order in `queue=` | Upload type first. [2] |

## Quick Recap

| Item | Meaning |
| --- | --- |
| `kind=pcq pcq-rate=` | Per-sub-stream limit [2] |
| `pcq-classifier=dst-address` / `src-address` | Per device, down / up [2] |
| `pcq-limit` | Queue size, not speed [2] |
| `queue=UP/DOWN` | Simple queue types [2] |

## Key Takeaways

- One PCQ queue replaces a queue per device.
- Classify downloads by destination and uploads by source.

## References

1. [RouterOS Manual — Queue Types](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/queue-types/)
2. [RouterOS Manual — PCQ example](https://manual.mikrotik.com/docs/firewall-and-quality-of-service/queues/pcq-example/)

---

**Previous:** [K.02 — Simple Queues](../02-simple-queues/README.md) · **Next:** K.04 — Queue Tree with Mangle

[Back to the main learning journal](../../README.md)
