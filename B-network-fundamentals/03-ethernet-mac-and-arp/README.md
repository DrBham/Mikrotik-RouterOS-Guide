<div align="center">

# Ethernet, MAC Addresses, and ARP

*Chapter B — Network Fundamentals & IP Addressing | Topic B.03*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Inside one local network, devices do not deliver frames by IP address. They use hardware (MAC) addresses. This topic explains MAC addresses, how IP travels inside Ethernet frames, how ARP connects an IP address to a MAC address, and how a bridge learns where each MAC address lives.

## Why This Matters

A.02 logged in by MAC address, A.03 put a laptop and a bridge on the same LAN, and every ping on that LAN starts with ARP. When "the IP is right but nothing answers", the ARP table is one of the first places to look.

## Prerequisites

| Topic | Why it is needed |
| --- | --- |
| [B.02 — OSI and TCP/IP Models](../02-osi-and-tcp-ip-models/README.md) | The link layer and encapsulation. |

## Theory

### MAC addresses

A globally unique 48-bit MAC address is also called an EUI-48 identifier. [1] The first 3 octets (24 bits) identify the organisation that was assigned the block, the OUI. [1] Two bits in the first octet have a special meaning [1]:

| Bit | 0 | 1 |
| --- | --- | --- |
| Group bit (lowest bit of the first octet) | Unicast: one device | Group: multicast or broadcast |
| Local bit | Globally administered by the prefix owner | Locally administered |

Documentation uses the reserved range `00:00:5E:00:53:00` to `00:00:5E:00:53:FF`, so examples never point at a real device. [1] This guide uses that range in every example.

### IP inside Ethernet

An IP datagram is carried inside an Ethernet frame whose type field is `0x0800`. [2] The largest IP datagram sent over Ethernet is 1500 octets. [2] This number is the Ethernet MTU you will see on every RouterOS Ethernet interface.
