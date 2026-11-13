# RouterOS Field Manual

**A Practical MikroTik Networking & Configuration Guide**

A community-oriented learning repository for MikroTik RouterOS, built through hands-on practice, original documentation, and verified configuration examples. The goal is to help networking students, MTCNA candidates, IT support technicians, and junior network engineers understand and reproduce common networking tasks, from fundamental administration to advanced networking.

> **Project status:** Initializing. The roadmap below describes planned coverage; it does not imply that tutorials have been implemented, tested, or published.

## Learning Approach

Each topic is developed through a practical learning cycle: understand the concept, plan the lab, implement the configuration, verify the results, document the findings, and publish the reviewed material.

Practical tutorials include two alternative configuration methods:

- **Method A — WinBox:** Step-by-step graphical instructions with original lab screenshots.
- **Method B — RouterOS CLI:** Equivalent terminal commands with explanations.
- **Verification and troubleshooting:** Expected behavior, observed test results, common failure scenarios, and recovery guidance where applicable.
- **Configuration file:** A topic-specific `.rsc` file reviewed for sensitive information and documented prerequisites.

Conceptual topics focus on clear explanations and relevant diagrams or screenshots where useful. They do not include unnecessary configuration files.

## Learning Roadmap

| Chapter | Topic |
| --- | --- |
| 01 | MikroTik Fundamentals |
| 02 | Network Fundamentals & IP Addressing |
| 03 | Basic Internet Connectivity |
| 04 | Interface & Bridge |
| 05 | DHCP & Network Services |
| 06 | Wireless Networking |
| 07 | Routing Fundamentals |
| 08 | Firewall & Network Security |
| 09 | Network Address Translation (NAT) |
| 10 | VLAN & Network Segmentation |
| 11 | Bandwidth Management & QoS |
| 12 | PPP & Hotspot |
| 13 | VPN & Network Tunneling |
| 14 | Load Balancing & High Availability |
| 15 | Dynamic Routing |
| 16 | IPv6 Networking |
| 17 | Network Monitoring & Troubleshooting |

The roadmap progresses from beginner concepts to intermediate implementation and advanced networking. Topics will be added incrementally as they are studied and validated.

## Repository Structure

Each chapter contains one folder per learning topic. A practical topic follows this layout:

```text
routeros-field-manual/
├── README.md
├── 01-mikrotik-fundamentals/
│   ├── 01-introduction-to-mikrotik/
│   │   └── README.md
│   └── 02-routeros-vs-routerboard/
│       └── README.md
└── 02-network-fundamentals/
    └── 08-static-ip-configuration/
        ├── README.md
        ├── images/
        │   ├── 01-open-ip-addresses.png
        │   ├── 02-add-ip-address.png
        │   └── 03-verify-configuration.png
        └── configuration.rsc
```

This is the planned structure, not a listing of completed tutorials. Topic folders and supporting files are created only when needed.

## Lab Environment

The project uses a MikroTik hAP lite (RB941-2nD) for physical labs and MikroTik Cloud Hosted Router (CHR) with GNS3 and VirtualBox for virtual labs. WinBox and the RouterOS CLI are used for configuration, with Wireshark available for packet analysis when needed.

Each practical tutorial identifies its actual device, RouterOS version, topology, prerequisites, and test environment. Hardware- or version-dependent features are documented accordingly.

## Documentation and Safety

Tutorials are based on original explanations and actual lab evidence. Untested configurations are explicitly identified; published examples are not a guarantee of suitability for production networks. Configuration changes should be reviewed and tested in an isolated environment before use. Sensitive credentials, private keys, and identifying device information must not be published.

## References

- [Official MikroTik RouterOS documentation](https://manual.mikrotik.com/)
- [MikroTik official website](https://mikrotik.com/)

## About This Project

RouterOS Field Manual is an independent learning and portfolio project. It is not affiliated with MikroTik and is not official certification training material. Its purpose is to build practical networking knowledge and share reproducible learning resources with the community.
