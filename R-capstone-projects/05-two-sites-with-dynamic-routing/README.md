<div align="center">

# Two Sites with VPN and Dynamic Routing

*Chapter R — Capstone Projects | Project R.05*

![Level](https://img.shields.io/badge/Level-Project-7C3AED) ![Type](https://img.shields.io/badge/Type-Capstone-2563EB) ![Status](https://img.shields.io/badge/Status-Planned-6B7280)

</div>

> [!NOTE]
> **Planned project.** This page describes the project for future work. The full build (design, CLI and WinBox walkthroughs, verification, and lab evidence) will be added after the topics it depends on have their lab screenshots.

## Goal

Join two offices over an encrypted tunnel and let OSPF exchange their routes, so adding a new subnet at either site needs no static routes.

## Topics this project combines

| Topic | What it contributes |
| --- | --- |
| M.02, M.06 | [WireGuard or IPsec](../../M-vpn-and-network-tunneling/02-wireguard-site-to-site/README.md) |
| M.05 | [GRE for routing protocols over IPsec](../../M-vpn-and-network-tunneling/05-eoip-and-gre-tunnels/README.md) |
| O.02–O.03 | [OSPF](../../O-dynamic-routing/02-single-area-ospf/README.md) |
| Q.06 | [Bandwidth test across the tunnel](../../Q-network-monitoring-and-troubleshooting/06-bandwidth-test/README.md) |

Every step of the build will reference these topics and their sources instead of repeating them.

## Planned contents

1. Requirements and the address plan, written as tables before any configuration.
2. Topology diagram (`images/topology.svg`).
3. Build order, one layer at a time, with a check after each step.
4. Method A — CLI, then reset, then Method B — WinBox, as in every topic.
5. Final verification and a troubleshooting table from real lab problems.
6. `configuration.rsc` export with secrets removed.

## Done when

- [ ] Tunnel up and encrypted
- [ ] Each site learns the other's subnets by OSPF
- [ ] New subnet appears at the other site automatically
- [ ] Tunnel throughput measured

---

**Previous:** [R.04](../04-dual-isp-office/README.md) · **Next:** End of the roadmap

[Back to the main learning journal](../../README.md)
