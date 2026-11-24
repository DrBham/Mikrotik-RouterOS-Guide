<div align="center">

# Dual-ISP Office

*Chapter R — Capstone Projects | Project R.04*

![Level](https://img.shields.io/badge/Level-Project-7C3AED) ![Type](https://img.shields.io/badge/Type-Capstone-2563EB) ![Status](https://img.shields.io/badge/Status-Planned-6B7280)

</div>

> [!NOTE]
> **Planned project.** This page describes the project for future work. The full build (design, CLI and WinBox walkthroughs, verification, and lab evidence) will be added after the topics it depends on have their lab screenshots.

## Goal

Use two internet connections together with PCC load balancing, fail over automatically when one breaks, send one server through a chosen line, and alert on outages.

## Topics this project combines

| Topic | What it contributes |
| --- | --- |
| N.01–N.02 | [Failover and PCC](../../N-load-balancing-and-high-availability/02-pcc-load-balancing/README.md) |
| G.02 | [Policy routing](../../G-routing-fundamentals/02-policy-routing/README.md) |
| N.04 | [Netwatch](../../N-load-balancing-and-high-availability/04-netwatch/README.md) |
| Q.01 | [Logs](../../Q-network-monitoring-and-troubleshooting/01-logs-and-remote-syslog/README.md) |

Every step of the build will reference these topics and their sources instead of repeating them.

## Planned contents

1. Requirements and the address plan, written as tables before any configuration.
2. Topology diagram (`images/topology.svg`).
3. Build order, one layer at a time, with a check after each step.
4. Method A — CLI, then reset, then Method B — WinBox, as in every topic.
5. Final verification and a troubleshooting table from real lab problems.
6. `configuration.rsc` export with secrets removed.

## Done when

- [ ] Connections split across both lines
- [ ] One line down: everything moves to the other
- [ ] Server always uses the chosen line
- [ ] Outage logged and alerted

---

**Previous:** [R.03](../03-community-wifi-hotspot/README.md) · **Next:** [R.05](../05-two-sites-with-dynamic-routing/README.md)

[Back to the main learning journal](../../README.md)
