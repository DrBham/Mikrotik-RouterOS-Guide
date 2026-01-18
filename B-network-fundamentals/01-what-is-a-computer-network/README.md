<div align="center">

# What Is a Computer Network

*Chapter B — Network Fundamentals & IP Addressing | Topic B.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Status](https://img.shields.io/badge/Status-Draft-F59E0B)

[Overview](#overview) · [Theory](#theory) · [In RouterOS](#how-it-appears-in-routeros) · [Quick Recap](#quick-recap) · [References](#references)

</div>

## Overview

Chapter A taught how to operate a MikroTik router. Chapter B explains what the router is actually doing. This first topic defines the pieces every later topic talks about: hosts, routers, networks, and the packets that travel between them.

## Why This Matters

When you configured A.03, the hAP lite joined two networks: your home Wi-Fi and your laptop's cable. Understanding why that needs a router, an address on each side, and NAT starts with knowing what a host, a network, and a router are.

## Prerequisites

None. This is the first theory topic.

## Theory

### Hosts

A host is the ultimate consumer of communication services: the computer that runs applications for its users, such as a laptop or a phone. [1] A host with more than one network interface is called multihomed. [1]

### Packet networks and routers

The internet is built from interconnected packet networks. The computers that join those networks together are called gateways or IP routers. [1] In OSI terms they are called intermediate systems. [1] This guide uses the word router, which is what RFC 1812 calls them. [2]

A router examines the IP header of each packet, removes the link-layer header, updates the IP header, and sends the packet out with a new link-layer header for the next network. [2]

### The internet is a network of networks

RFC 1122 states the idea directly: the internet is a network of networks. A host is attached to one particular network; its connection to the wider internet exists only through routers. [1]

### Datagrams

IP sends data in independent units called datagrams. IP delivers each datagram from a source to a destination across interconnected networks, but it does not guarantee delivery, order, or flow control. [3] Routers do not keep connection state; each datagram is forwarded on its own. [1] End-to-end state, such as which data has arrived, lives in the hosts. [1]

### Where the work happens

RFC 1122's design assumptions put routing complexity in the routers and keep hosts simpler, and they expect the network to tolerate wide variation in bandwidth, delay, loss, and packet size. [1]

### How it connects

```text
B.01 hosts, routers, networks
  └─► B.02 layers describe how a packet is built
       └─► B.03–B.07 how hosts on one network and across networks find each other
            └─► B.08–B.10 the services that make the internet usable (DNS, DHCP, NAT)
```
