# Detailed Roadmap

This file maps every planned topic folder, its type, and the sources each topic must be written from. The chapter list matches the [main README](README.md#learning-roadmap).

**Status legend:** ✅ published · 📝 draft (waiting for lab screenshots) · ⏳ planned

**Topic types**

| Type | Contents |
| :--- | :--- |
| Theory | Concepts, why they matter, how they connect to other topics, cited sources. No configuration file. |
| Practice | Short theory recap, Method A (WinBox, screenshots), Method B (CLI, screenshots), verification, troubleshooting, `configuration.rsc`. |
| Theory + Practice | Both of the above in one folder. |

## Learning path

The first three chapters are meant to be followed in order:

1. **Chapter A** teaches what RouterOS is and how to log in to it.
2. **Chapter B** teaches the network theory needed to understand what the router is doing.
3. **Chapter C** uses A and B to connect a MikroTik router to the internet.

Everything after Chapter C builds on a router that already has internet access.

## Source policy

Every theory statement must be traceable to a source listed in the topic's **References** section:

1. **Primary:** the official RouterOS manual at [manual.mikrotik.com](https://manual.mikrotik.com/) (the old [help.mikrotik.com](https://help.mikrotik.com/docs/) site is frozen; cite it only if the new manual has no equivalent page).
2. **Protocol standards:** IETF RFCs at [rfc-editor.org](https://www.rfc-editor.org/) for general networking concepts (IP, DHCP, DNS, NAT, ARP).
3. **Product pages:** [mikrotik.com](https://mikrotik.com/) for hardware facts.
4. **Lab evidence:** my own screenshots and command output, labelled as observations, not as general facts.

Statements that cannot yet be traced to a source are marked `> [!WARNING] Source needed` and must not be published as fact.

---

## A — MikroTik Fundamentals

| # | Folder | Type | Status | Main sources |
| :-- | :--- | :--- | :---: | :--- |
| 01 | `01-introduction-to-mikrotik` | Theory | 📝 | mikrotik.com/software, CHR Installation, Management Tools |
| 02 | `02-connecting-to-routeros-using-winbox` | Practice | 📝 | WinBox, MAC server, First Time Configuration (Accessing the router) |
| 03 | `03-routeros-cli-basics` | Practice | ⏳ | Console |
| 04 | `04-default-configuration-and-reset` | Theory + Practice | ⏳ | Default configurations, RouterOS configuration reset |
| 05 | `05-users-and-basic-router-security` | Practice | ⏳ | Securing your router, User management |
| 06 | `06-backup-export-and-restore` | Practice | ⏳ | Configuration management (backup/export) |
| 07 | `07-upgrading-routeros-and-packages` | Practice | ⏳ | Upgrading and installation, Packages |

## B — Network Fundamentals & IP Addressing

Mostly theory. Each topic ends with a short "How this appears in RouterOS" section so the theory is tied to the router.

| # | Folder | Type | Status | Main sources |
| :-- | :--- | :--- | :---: | :--- |
| 01 | `01-what-is-a-computer-network` | Theory | ⏳ | RFC 1122 (host requirements, layering) |
| 02 | `02-osi-and-tcp-ip-models` | Theory | ⏳ | RFC 1122, ISO/IEC 7498-1 (OSI) |
| 03 | `03-ethernet-mac-and-arp` | Theory | ⏳ | RFC 826 (ARP), RouterOS ARP page |
| 04 | `04-ipv4-addresses-and-subnet-masks` | Theory | ⏳ | RFC 791 (IPv4), RouterOS IP Addressing |
| 05 | `05-subnetting-and-cidr` | Theory | ⏳ | RFC 4632 (CIDR) |
| 06 | `06-public-and-private-ip-addresses` | Theory | ⏳ | RFC 1918, RFC 6598 |
| 07 | `07-gateway-and-default-route` | Theory | ⏳ | RFC 1812, RouterOS IP Routing |
| 08 | `08-dns-concepts` | Theory | ⏳ | RFC 1034, RFC 1035, RouterOS DNS |
| 09 | `09-dhcp-concepts` | Theory | ⏳ | RFC 2131, RouterOS DHCP |
| 10 | `10-nat-concepts` | Theory | ⏳ | RFC 3022, RouterOS NAT |
| 11 | `11-static-ip-configuration` | Practice | ⏳ | RouterOS IP Addressing |

## C — Basic Internet Connectivity

This is the first full lab: a router with no configuration becomes an internet gateway for a LAN. It follows the official [First Time Configuration](https://manual.mikrotik.com/docs/getting-started/first-time-configuration) guide, with each step explained using Chapter B theory.

| # | Folder | Type | Status | Main sources |
| :-- | :--- | :--- | :---: | :--- |
| 01 | `01-lab-topology-and-planning` | Theory | ⏳ | First Time Configuration |
| 02 | `02-wan-dhcp-client` | Practice | ⏳ | First Time Configuration, DHCP client |
| 03 | `03-wan-static-ip-route-and-dns` | Practice | ⏳ | First Time Configuration, IP address, DNS |
| 04 | `04-wan-pppoe-client` | Practice | ⏳ | First Time Configuration, PPPoE |
| 05 | `05-lan-bridge-ip-and-dhcp-server` | Practice | ⏳ | Bridging and Switching, DHCP server |
| 06 | `06-nat-masquerade` | Practice | ⏳ | First Time Configuration, NAT |
| 07 | `07-verify-and-troubleshoot-connectivity` | Practice | ⏳ | Ping, Traceroute, Torch |

## D – Q — Later chapters

Topic lists are planned when Chapter C is complete.

| Chapter | Title |
| :-- | :--- |
| D | Interface & Bridge |
| E | DHCP & Network Services |
| F | Wireless Networking |
| G | Routing Fundamentals |
| H | Firewall & Network Security |
| I | Network Address Translation (NAT) |
| J | VLAN & Network Segmentation |
| K | Bandwidth Management & QoS |
| L | PPP & Hotspot |
| M | VPN & Network Tunneling |
| N | Load Balancing & High Availability |
| O | Dynamic Routing |
| P | IPv6 Networking |
| Q | Network Monitoring & Troubleshooting |
