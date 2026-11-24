<div align="center">

# Small Office Network

*Chapter R — Capstone Projects | Project R.02*

![Level](https://img.shields.io/badge/Level-Project-7C3AED) ![Type](https://img.shields.io/badge/Type-Capstone-2563EB) ![Status](https://img.shields.io/badge/Status-Planned-6B7280)

</div>

> [!NOTE]
> **Planned project.** This page describes the project for future work. The full build (design, CLI and WinBox walkthroughs, verification, and lab evidence) will be added after the topics it depends on have their lab screenshots.

## Goal

Separate staff, guests, and devices with VLANs on one router and a managed switch, enforce an inter-VLAN policy, and let staff work from home through a VPN.

## Topics this project combines

| Topic | What it contributes |
| --- | --- |
| J | [VLAN & Network Segmentation](../../J-vlan-and-network-segmentation/01-vlan-concepts/README.md) |
| H | [Firewall](../../H-firewall-and-network-security/01-firewall-concepts-and-packet-flow/README.md) |
| M.02–M.03 | [WireGuard](../../M-vpn-and-network-tunneling/03-wireguard-road-warrior/README.md) |
| Q.01–Q.03 | [Logs, graphs, SNMP](../../Q-network-monitoring-and-troubleshooting/01-logs-and-remote-syslog/README.md) |

Every step of the build will reference these topics and their sources instead of repeating them.

## Planned contents

1. Requirements and the address plan, written as tables before any configuration.
2. Topology diagram (`images/topology.svg`).
3. Build order, one layer at a time, with a check after each step.
4. Method A — CLI, then reset, then Method B — WinBox, as in every topic.
5. Final verification and a troubleshooting table from real lab problems.
6. `configuration.rsc` export with secrets removed.

## Done when

- [ ] Each VLAN gets its own subnet and DHCP
- [ ] Guests reach only the internet
- [ ] Remote staff reach the office LAN
- [ ] Logs kept on a syslog server

---

**Previous:** [R.01](../01-complete-home-network/README.md) · **Next:** [R.03](../03-community-wifi-hotspot/README.md)

[Back to the main learning journal](../../README.md)
