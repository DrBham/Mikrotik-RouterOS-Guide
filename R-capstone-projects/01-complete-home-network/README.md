<div align="center">

# Complete Home Network

*Chapter R — Capstone Projects | Project R.01*

![Level](https://img.shields.io/badge/Level-Project-7C3AED) ![Type](https://img.shields.io/badge/Type-Capstone-2563EB) ![Status](https://img.shields.io/badge/Status-Planned-6B7280)

</div>

> [!NOTE]
> **Planned project.** This page describes the project for future work. The full build (design, CLI and WinBox walkthroughs, verification, and lab evidence) will be added after the topics it depends on have their lab screenshots.

## Goal

Build a home router from an empty configuration: internet over Wi-Fi or cable, LAN with DHCP and local DNS, a secured Wi-Fi network, a guest network, the input and forward firewall, ad blocking, and a nightly backup.

## Topics this project combines

| Topic | What it contributes |
| --- | --- |
| C | [Basic Internet Connectivity](../../C-basic-internet-connectivity/01-lab-topology-and-planning/README.md) |
| E.03, E.06, E.07 | [DNS entries, scheduled backup, adlist](../../E-dhcp-and-network-services/07-dns-adlist/README.md) |
| F.01–F.04 | [Wi-Fi AP, WPA2, guest SSID](../../F-wireless-networking/04-virtual-ap-guest-network/README.md) |
| H.02–H.03 | [Firewall](../../H-firewall-and-network-security/02-protecting-the-router/README.md) |

Every step of the build will reference these topics and their sources instead of repeating them.

## Planned contents

1. Requirements and the address plan, written as tables before any configuration.
2. Topology diagram (`images/topology.svg`).
3. Build order, one layer at a time, with a check after each step.
4. Method A — CLI, then reset, then Method B — WinBox, as in every topic.
5. Final verification and a troubleshooting table from real lab problems.
6. `configuration.rsc` export with secrets removed.

## Done when

- [ ] Laptop and phone reach the internet
- [ ] Guest network cannot reach the LAN
- [ ] Router unreachable from the WAN
- [ ] Backup file created every night

---

**Previous:** [Q.06 — Bandwidth Test](../../Q-network-monitoring-and-troubleshooting/06-bandwidth-test/README.md) · **Next:** [R.02](../02-small-office-network/README.md)

[Back to the main learning journal](../../README.md)
