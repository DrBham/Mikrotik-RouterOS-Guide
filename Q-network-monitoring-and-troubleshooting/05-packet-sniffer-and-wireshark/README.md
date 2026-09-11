<div align="center">

# Packet Sniffer and Wireshark

*Chapter Q — Network Monitoring & Troubleshooting | Topic Q.05*

![Level](https://img.shields.io/badge/Level-Advanced-7C3AED) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [Method A — CLI](#method-a--cli) · [Method B — WinBox](#method-b--winbox-gui) · [Quick Recap](#quick-recap) · [References](#references)

</div>

> [!NOTE]
> **Draft.** Screenshots and lab results are not recorded yet. Every `TODO` must be replaced with real lab evidence before publishing.

## Overview

When logs and counters cannot explain a problem, look at the packets. RouterOS has a built-in packet sniffer that can show packets live, save them to a file, or stream them to Wireshark on the laptop. This topic captures a DHCP exchange three ways.

## Why This Matters

Seeing the actual DORA exchange (B.09), a failing TCP handshake, or a missing DNS reply ends guessing. Capturing on the router also sees traffic of devices you cannot install Wireshark on.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.09 — DHCP Concepts](../../B-network-fundamentals/09-dhcp-concepts/README.md) | What to look for in the capture. |
| [D.04 — Port Mirroring with the Switch Chip](../../D-interface-and-bridge/04-port-mirroring-with-the-switch-chip/README.md) | The hardware alternative. |

## Theory

### Three ways to capture

| Mode | Command [1] |
| --- | --- |
| Live view | `/tool/sniffer/quick interface=ether1 port=bootps,bootpc duration=10s` |
| Save to file | `/tool/sniffer/start …`, `/tool/sniffer/stop`, `/tool/sniffer/save file-name=capture.pcap` |
| Stream to Wireshark | `/tool/sniffer/set streaming-enabled=yes streaming-server=…` then `start` |

Streaming uses TZSP on UDP 37008; in Wireshark, filter with `udp.port==37008`. [1] From RouterOS 7.20, saved files are pcapng. [1]

### Cost

While running, the sniffer disables the IPv4 fast path. [1] On a small router, capture briefly and with filters.

### Privacy

Captures contain addresses, host names, and sometimes unencrypted content. Do not publish raw capture files.

## Lab Environment

| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite, `TODO` version, default configuration |
| Laptop | `192.168.88.50` with Wireshark |

## Method A — CLI

### Step 1 — Live view of DHCP

```routeros
/tool/sniffer/quick interface=bridge port=bootps,bootpc duration=10s
```

[1] While it runs, `ipconfig /renew` on the laptop.

> [!CAUTION]
> **Privacy:** cover MAC addresses and host names. See the [privacy checklist](../../TOPIC-TEMPLATE.md#screenshot--privacy-checklist).

`TODO` ![Step 1](images/cli/01-quick.png)

### Step 2 — Save to a file

```routeros
/tool/sniffer/start interface=bridge ip-address=192.168.88.50/32
/tool/sniffer/stop
/tool/sniffer/save file-name=capture.pcap
```

[1] Download `capture.pcap` (A.07) and open it in Wireshark.

`TODO` ![Step 2](images/cli/02-file.png)

### Step 3 — Stream to Wireshark

```routeros
/tool/sniffer/set streaming-enabled=yes streaming-server=192.168.88.50 filter-stream=yes
/tool/sniffer/start interface=ether1
```

[1] In Wireshark, capture on the laptop's Ethernet with filter `udp.port==37008`. Stop with `/tool/sniffer/stop`.

`TODO` ![Step 3](images/cli/03-stream.png)

### Verify (CLI)

| Check | Expected result | Actual result |
| --- | --- | --- |
| Live view | DISCOVER, OFFER, REQUEST, ACK | `TODO` |
| Saved file | Opens in Wireshark | `TODO` |
| Stream | Packets arrive in Wireshark | `TODO` |
