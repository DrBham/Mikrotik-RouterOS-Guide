<div align="center">

# OSI and TCP/IP Models

*Chapter B — Network Fundamentals & IP Addressing | Topic B.02*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Network protocols are described in layers. Each layer solves one part of the problem of moving data between programs on different computers, and uses the layer below it. This topic introduces the two layer models you will meet everywhere: the four-layer Internet (TCP/IP) model and the seven-layer OSI reference model.

## Why This Matters

Troubleshooting is easier when you can say which layer is failing. "The cable is unplugged", "the laptop has no IP address", "the port is blocked", and "the website name does not resolve" are problems at different layers, and RouterOS puts the tools for each in different menus.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.01 — What Is a Computer Network](../01-what-is-a-computer-network/README.md) | Hosts, routers, and packets. |

## Theory

### The Internet protocol suite: four layers

RFC 1122 describes the Internet protocol suite as four layers. [1]

| Layer | What it does [1] | Examples [1] |
| --- | --- | --- |
| Application | The top layer. The Internet suite does not split it further; it covers what OSI calls the presentation and application layers. | Telnet, FTP, SMTP, DNS |
| Transport | End-to-end communication between applications. | TCP (reliable, connection oriented), UDP (connectionless datagrams) |
| Internet | Carries datagrams from the source host to the destination host. Includes addressing and fragmentation. | IP, ICMP, IGMP |
| Link | The protocol used on the directly connected network. | Ethernet |

IP itself is connectionless and does not guarantee end-to-end delivery. [1] Reliability, when an application needs it, comes from TCP in the transport layer. [1]

### The OSI reference model: seven layers

ITU-T Recommendation X.200 defines the Open Systems Interconnection (OSI) Basic Reference Model. Clause 7 describes its seven layers, from top to bottom [2]:

| # | OSI layer [2] | Closest Internet layer |
| --- | --- | --- |
| 7 | Application | Application [1] |
| 6 | Presentation | Application [1] |
| 5 | Session | Application |
| 4 | Transport | Transport |
| 3 | Network | Internet |
| 2 | Data Link | Link |
| 1 | Physical | Link |

> [!WARNING]
> **Source needed:** the one-line function of each OSI layer from X.200 clauses 7.1 to 7.7, and an official source for the mapping of Session, Data Link, and Physical in the last column. RFC 1122 only states the Application/Presentation mapping. [1]

The OSI model is a reference for naming layers. The protocols actually used on the internet follow the four-layer suite above. [1]
