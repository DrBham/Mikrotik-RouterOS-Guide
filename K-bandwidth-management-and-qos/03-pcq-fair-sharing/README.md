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
