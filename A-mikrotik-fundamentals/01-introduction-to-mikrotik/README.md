<div align="center">

# Introduction to MikroTik

*Chapter 01 — MikroTik Fundamentals | Topic 01.01*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Theory-2563EB) ![Topic](https://img.shields.io/badge/Topic-RouterOS-0891B2)

[Overview](#overview) · [Core Concepts](#core-concepts) · [Management Tools](#management-tools) · [Key Takeaways](#key-takeaways) · [References](#references)

</div>

## Overview

Before learning how to configure a MikroTik router, it helps to understand what MikroTik is, what RouterOS does, and how we interact with it. These names often appear together, but they refer to different things.

This is the first topic in my RouterOS learning journal. I want to build a clear understanding of the platform before moving on to actual configuration.

| What we will learn | Why it matters |
| --- | --- |
| MikroTik, RouterOS, and RouterBOARD | Understand the difference between the company, software, and hardware. |
| Cloud Hosted Router (CHR) | Understand how RouterOS can be used in a virtual lab. |
| Management tools | Know how to access a router and why closing a management application does not stop the router. |

## Core Concepts

### 1. What is MikroTik?

MikroTik is a company that develops networking hardware and the RouterOS software that runs on it. According to its company page, product development takes place in the EU. [1]

> [!WARNING]
> **Source needed:** founding year and headquarters city. The company page [1] does not state them; the linked company profile has not been checked yet.

The name MikroTik is often used to refer to a router, but the company also develops switches, wireless devices, and the operating system used to manage network functions.

### 2. RouterOS and RouterBOARD

RouterOS is MikroTik's stand-alone operating system, based on the Linux kernel, and it powers MikroTik hardware devices. Its features include routing, firewall, bandwidth management, wireless access point, hotspot gateway, and VPN server functions. [2]

RouterBOARD is the name of MikroTik's hardware line; it is not another name for the operating system.

> [!WARNING]
> **Source needed:** an official page that defines RouterBOARD as MikroTik's hardware platform. [2] lists it only as a product category.

| Component | What it means |
| --- | --- |
| MikroTik | The company that develops networking products. |
| RouterBOARD | MikroTik network hardware. |
| RouterOS | The operating system that runs network services and processes traffic. |
| WinBox | A graphical application used to manage RouterOS. |

For comparison, a computer runs an operating system and applications allow us to interact with it. Similarly, a MikroTik router runs RouterOS, while WinBox provides a graphical way to configure it.

### 3. What is Cloud Hosted Router (CHR)?

Cloud Hosted Router (CHR) is a virtualized edition of RouterOS designed to run on x86_64 virtual environments. [3] It lets me practise RouterOS without a separate physical router.

In my learning environment, I use a physical hAP lite and virtual RouterOS instances for different kinds of exercises.

| Physical lab | Virtual lab |
| --- | --- |
| MikroTik hAP lite (RB941-2nD) | MikroTik CHR |
| Physical Ethernet ports and built-in Wi-Fi radio | Virtual network interfaces; no built-in physical Wi-Fi radio |
| Resources limited by the device hardware | Resources depend on VM allocation and platform limits |
| Useful for hands-on cabling and wireless exercises | Useful for multi-router topologies in GNS3 |

Both environments support core routing and firewall concepts. However, not every hardware-dependent feature can be reproduced in CHR. RouterOS versions, installed packages, and device capabilities must be checked before following a configuration guide. [2][3][4]

## Management Tools

RouterOS can be managed in several ways. [5]

| Tool | How it is used |
| --- | --- |
| WinBox | Graphical application for administering RouterOS. WinBox v4 runs natively on Windows, Linux, and macOS. [7] |
| WebFig | Browser-based management interface. |
| Console (CLI) | Text-based access over serial, Telnet, SSH, or the WinBox terminal. [5] |

SSH and WebFig require the relevant service to be enabled and reachable, along with valid authentication. Management services should not be exposed directly to the public internet. [6]

### Does closing WinBox stop the router?

No. This was an important distinction for me while studying RouterOS. WinBox is a management application running on my computer; RouterOS runs on the router itself.

For example, if I configure a DHCP server on my hAP lite and then close WinBox, another laptop connected to the correct LAN can still receive an IP address, provided the DHCP service is working. Closing the management application does not stop that service.

Receiving an IP address is not the same as having internet access. Internet connectivity also depends on the upstream connection, routing, firewall policy, and, where needed, NAT and DNS.

## Lab Environment

| Physical environment | Virtual environment |
| --- | --- |
| MikroTik hAP lite (RB941-2nD) | GNS3 and MikroTik CHR |
| Ethernet connection and WinBox | VirtualBox and virtual network interfaces |

This topic is a theory introduction. No configuration changes or new lab tests are required, and the example above illustrates expected behavior rather than a recorded test result. The actual RouterOS version and test environment will be recorded in each practical tutorial.

## Quick Recap

| Term | Meaning |
| --- | --- |
| MikroTik | The company. |
| RouterOS | The operating system on MikroTik devices. |
| RouterBOARD | MikroTik hardware. |
| CHR | RouterOS for virtual machines (x86_64). |
| WinBox / WebFig / Console | Ways to manage RouterOS, not RouterOS itself. |

## Key Takeaways

- MikroTik is the company; RouterOS is the network operating system.
- RouterBOARD is the hardware platform, while CHR is a virtual edition of RouterOS.
- WinBox, WebFig, and SSH provide ways to manage RouterOS; they are not the operating system itself.
- Closing a management application does not stop normal router operation.
- Physical and virtual labs are both useful, but their capabilities are not always identical.

## References

1. [MikroTik — Company](https://mikrotik.com/aboutus/company)
2. [MikroTik — RouterOS](https://mikrotik.com/software)
3. [RouterOS Manual — CHR Installation](https://manual.mikrotik.com/docs/getting-started/installation-and-upgrade/install/chr-installation/)
4. [MikroTik — hAP lite](https://mikrotik.com/product/RB941-2nD)
5. [RouterOS Manual — Management Tools](https://manual.mikrotik.com/docs/management-tools/)
6. [RouterOS Manual — Securing Your Router](https://manual.mikrotik.com/docs/getting-started/securing-your-router/)
7. [RouterOS Manual — WinBox](https://manual.mikrotik.com/docs/management-tools/winbox/)

---

**Next topic:** [A.02 — Connecting to RouterOS Using WinBox](../02-connecting-to-routeros-using-winbox/README.md) (draft)

[Back to the main learning journal](../../README.md)
