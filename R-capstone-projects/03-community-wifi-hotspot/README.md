<div align="center">

# Community Wi-Fi with HotSpot and PPPoE

*Chapter R — Capstone Projects | Project R.03*

![Level](https://img.shields.io/badge/Level-Project-7C3AED) ![Type](https://img.shields.io/badge/Type-Capstone-2563EB) ![Status](https://img.shields.io/badge/Status-Planned-6B7280)

</div>

> [!NOTE]
> **Planned project.** This page describes the project for future work. The full build (design, CLI and WinBox walkthroughs, verification, and lab evidence) will be added after the topics it depends on have their lab screenshots.

## Goal

Run a small neighbourhood network: HotSpot vouchers for visitors, PPPoE accounts for subscribers, fair sharing of the uplink, and basic monitoring.

## Topics this project combines

| Topic | What it contributes |
| --- | --- |
| L | [PPP & Hotspot](../../L-ppp-and-hotspot/01-pppoe-server-profiles-and-secrets/README.md) |
| K.02–K.03 | [Simple queues and PCQ](../../K-bandwidth-management-and-qos/03-pcq-fair-sharing/README.md) |
| I | [NAT](../../I-network-address-translation/01-port-forwarding-with-dst-nat/README.md) |
| Q | [Monitoring](../../Q-network-monitoring-and-troubleshooting/02-resources-profiler-and-graphing/README.md) |

Every step of the build will reference these topics and their sources instead of repeating them.

## Planned contents

1. Requirements and the address plan, written as tables before any configuration.
2. Topology diagram (`images/topology.svg`).
3. Build order, one layer at a time, with a check after each step.
4. Method A — CLI, then reset, then Method B — WinBox, as in every topic.
5. Final verification and a troubleshooting table from real lab problems.
6. `configuration.rsc` export with secrets removed.

## Done when

- [ ] Voucher users limited by time and speed
- [ ] PPPoE users get their profile limits
- [ ] No user can take the whole uplink
- [ ] Active users visible at a glance

---

**Previous:** [R.02](../02-small-office-network/README.md) · **Next:** [R.04](../04-dual-isp-office/README.md)

[Back to the main learning journal](../../README.md)
