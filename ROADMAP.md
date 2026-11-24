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

1. **Chapter A** teaches what RouterOS is, how to log in to it, and how to give the lab router internet access (A.03) so the guide stays reachable while practising.
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
| 03 | `03-lab-internet-access` | Theory + Practice | 📝 | Default configurations, Wireless Interface, DHCP Client, First Time Configuration |
| 04 | `04-routeros-cli-basics` | Practice | 📝 | Console |
| 05 | `05-default-configuration-and-reset` | Theory + Practice | 📝 | Default configurations, RouterOS configuration reset |
| 06 | `06-users-and-basic-router-security` | Practice | 📝 | Securing your router, User management |
| 07 | `07-backup-export-and-restore` | Practice | 📝 | Configuration management (backup/export) |
| 08 | `08-upgrading-routeros-and-packages` | Practice | 📝 | Upgrading and installation, Packages |

## B — Network Fundamentals & IP Addressing

Mostly theory. Each topic ends with a short "How this appears in RouterOS" section so the theory is tied to the router.

| # | Folder | Type | Status | Main sources |
| :-- | :--- | :--- | :---: | :--- |
| 01 | `01-what-is-a-computer-network` | Theory | 📝 | RFC 1122 (host requirements, layering) |
| 02 | `02-osi-and-tcp-ip-models` | Theory | 📝 | RFC 1122, ISO/IEC 7498-1 (OSI) |
| 03 | `03-ethernet-mac-and-arp` | Theory | 📝 | RFC 826 (ARP), RouterOS ARP page |
| 04 | `04-ipv4-addresses-and-subnet-masks` | Theory | 📝 | RFC 791 (IPv4), RouterOS IP Addressing |
| 05 | `05-subnetting-and-cidr` | Theory | 📝 | RFC 4632 (CIDR) |
| 06 | `06-public-and-private-ip-addresses` | Theory | 📝 | RFC 1918, RFC 6598 |
| 07 | `07-gateway-and-default-route` | Theory | 📝 | RFC 1812, RouterOS IP Routing |
| 08 | `08-dns-concepts` | Theory | 📝 | RFC 1034, RFC 1035, RouterOS DNS |
| 09 | `09-dhcp-concepts` | Theory | 📝 | RFC 2131, RouterOS DHCP |
| 10 | `10-nat-concepts` | Theory | 📝 | RFC 3022, RouterOS NAT |
| 11 | `11-static-ip-configuration` | Practice | 📝 | RouterOS IP Addressing |

## C — Basic Internet Connectivity

This is the first full lab: a router with no configuration becomes an internet gateway for a LAN. It follows the official [First Time Configuration](https://manual.mikrotik.com/docs/getting-started/first-time-configuration) guide, with each step explained using Chapter B theory.

| # | Folder | Type | Status | Main sources |
| :-- | :--- | :--- | :---: | :--- |
| 01 | `01-lab-topology-and-planning` | Theory | 📝 | First Time Configuration |
| 02 | `02-wan-dhcp-client` | Practice | 📝 | First Time Configuration, DHCP client |
| 03 | `03-wan-static-ip-route-and-dns` | Practice | 📝 | First Time Configuration, IP address, DNS |
| 04 | `04-wan-pppoe-client` | Practice | 📝 | First Time Configuration, PPPoE |
| 05 | `05-lan-bridge-ip-and-dhcp-server` | Practice | 📝 | Bridging and Switching, DHCP server |
| 06 | `06-nat-masquerade` | Practice | 📝 | First Time Configuration, NAT |
| 07 | `07-verify-and-troubleshoot-connectivity` | Practice | 📝 | Ping, Traceroute, Torch |

## D – Q — Topic lists

All topics below are drafted (📝). Sources are listed in each topic's References section.

### D — Interface & Bridge

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-ethernet-interfaces-and-mtu`](D-interface-and-bridge/01-ethernet-interfaces-and-mtu/README.md) | 📝 |
| 02 | [`02-bridge-ports-and-host-table`](D-interface-and-bridge/02-bridge-ports-and-host-table/README.md) | 📝 |
| 03 | [`03-spanning-tree-protocol`](D-interface-and-bridge/03-spanning-tree-protocol/README.md) | 📝 |
| 04 | [`04-port-mirroring-with-the-switch-chip`](D-interface-and-bridge/04-port-mirroring-with-the-switch-chip/README.md) | 📝 |
| 05 | [`05-bonding`](D-interface-and-bridge/05-bonding/README.md) | 📝 |

### E — DHCP & Network Services

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-dhcp-static-leases-and-options`](E-dhcp-and-network-services/01-dhcp-static-leases-and-options/README.md) | 📝 |
| 02 | [`02-dhcp-relay`](E-dhcp-and-network-services/02-dhcp-relay/README.md) | 📝 |
| 03 | [`03-dns-static-entries-and-forwarding`](E-dhcp-and-network-services/03-dns-static-entries-and-forwarding/README.md) | 📝 |
| 04 | [`04-ntp-clock-and-identity`](E-dhcp-and-network-services/04-ntp-clock-and-identity/README.md) | 📝 |
| 05 | [`05-ip-cloud-and-ddns`](E-dhcp-and-network-services/05-ip-cloud-and-ddns/README.md) | 📝 |
| 06 | [`06-scheduler-and-scripts`](E-dhcp-and-network-services/06-scheduler-and-scripts/README.md) | 📝 |
| 07 | [`07-dns-adlist`](E-dhcp-and-network-services/07-dns-adlist/README.md) | 📝 |
| 08 | [`08-back-to-home`](E-dhcp-and-network-services/08-back-to-home/README.md) | 📝 |

### F — Wireless Networking

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-wireless-ap-basics`](F-wireless-networking/01-wireless-ap-basics/README.md) | 📝 |
| 02 | [`02-wireless-security-profiles`](F-wireless-networking/02-wireless-security-profiles/README.md) | 📝 |
| 03 | [`03-access-list-and-connect-list`](F-wireless-networking/03-access-list-and-connect-list/README.md) | 📝 |
| 04 | [`04-virtual-ap-guest-network`](F-wireless-networking/04-virtual-ap-guest-network/README.md) | 📝 |
| 05 | [`05-capsman-legacy`](F-wireless-networking/05-capsman-legacy/README.md) | 📝 |

### G — Routing Fundamentals

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-static-routes`](G-routing-fundamentals/01-static-routes/README.md) | 📝 |
| 02 | [`02-policy-routing`](G-routing-fundamentals/02-policy-routing/README.md) | 📝 |

### H — Firewall & Network Security

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-firewall-concepts-and-packet-flow`](H-firewall-and-network-security/01-firewall-concepts-and-packet-flow/README.md) | 📝 |
| 02 | [`02-protecting-the-router`](H-firewall-and-network-security/02-protecting-the-router/README.md) | 📝 |
| 03 | [`03-protecting-the-lan`](H-firewall-and-network-security/03-protecting-the-lan/README.md) | 📝 |
| 04 | [`04-address-lists-and-raw-rules`](H-firewall-and-network-security/04-address-lists-and-raw-rules/README.md) | 📝 |
| 05 | [`05-brute-force-protection`](H-firewall-and-network-security/05-brute-force-protection/README.md) | 📝 |

### I — Network Address Translation (NAT)

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-port-forwarding-with-dst-nat`](I-network-address-translation/01-port-forwarding-with-dst-nat/README.md) | 📝 |
| 02 | [`02-hairpin-nat`](I-network-address-translation/02-hairpin-nat/README.md) | 📝 |
| 03 | [`03-redirect-netmap-and-nat-helpers`](I-network-address-translation/03-redirect-netmap-and-nat-helpers/README.md) | 📝 |

### J — VLAN & Network Segmentation

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-vlan-concepts`](J-vlan-and-network-segmentation/01-vlan-concepts/README.md) | 📝 |
| 02 | [`02-vlan-interfaces-on-a-router`](J-vlan-and-network-segmentation/02-vlan-interfaces-on-a-router/README.md) | 📝 |
| 03 | [`03-bridge-vlan-filtering`](J-vlan-and-network-segmentation/03-bridge-vlan-filtering/README.md) | 📝 |
| 04 | [`04-inter-vlan-routing-and-isolation`](J-vlan-and-network-segmentation/04-inter-vlan-routing-and-isolation/README.md) | 📝 |

### K — Bandwidth Management & QoS

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-queue-concepts-and-htb`](K-bandwidth-management-and-qos/01-queue-concepts-and-htb/README.md) | 📝 |
| 02 | [`02-simple-queues`](K-bandwidth-management-and-qos/02-simple-queues/README.md) | 📝 |
| 03 | [`03-pcq-fair-sharing`](K-bandwidth-management-and-qos/03-pcq-fair-sharing/README.md) | 📝 |
| 04 | [`04-queue-tree-with-mangle`](K-bandwidth-management-and-qos/04-queue-tree-with-mangle/README.md) | 📝 |
| 05 | [`05-kid-control`](K-bandwidth-management-and-qos/05-kid-control/README.md) | 📝 |

### L — PPP & Hotspot

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-pppoe-server-profiles-and-secrets`](L-ppp-and-hotspot/01-pppoe-server-profiles-and-secrets/README.md) | 📝 |
| 02 | [`02-hotspot-setup`](L-ppp-and-hotspot/02-hotspot-setup/README.md) | 📝 |
| 03 | [`03-hotspot-users-and-profiles`](L-ppp-and-hotspot/03-hotspot-users-and-profiles/README.md) | 📝 |

### M — VPN & Network Tunneling

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-vpn-and-tunnel-concepts`](M-vpn-and-network-tunneling/01-vpn-and-tunnel-concepts/README.md) | 📝 |
| 02 | [`02-wireguard-site-to-site`](M-vpn-and-network-tunneling/02-wireguard-site-to-site/README.md) | 📝 |
| 03 | [`03-wireguard-road-warrior`](M-vpn-and-network-tunneling/03-wireguard-road-warrior/README.md) | 📝 |
| 04 | [`04-l2tp-over-ipsec`](M-vpn-and-network-tunneling/04-l2tp-over-ipsec/README.md) | 📝 |
| 05 | [`05-eoip-and-gre-tunnels`](M-vpn-and-network-tunneling/05-eoip-and-gre-tunnels/README.md) | 📝 |
| 06 | [`06-ipsec-site-to-site`](M-vpn-and-network-tunneling/06-ipsec-site-to-site/README.md) | 📝 |

### N — Load Balancing & High Availability

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-wan-failover`](N-load-balancing-and-high-availability/01-wan-failover/README.md) | 📝 |
| 02 | [`02-pcc-load-balancing`](N-load-balancing-and-high-availability/02-pcc-load-balancing/README.md) | 📝 |
| 03 | [`03-vrrp-gateway-redundancy`](N-load-balancing-and-high-availability/03-vrrp-gateway-redundancy/README.md) | 📝 |
| 04 | [`04-netwatch`](N-load-balancing-and-high-availability/04-netwatch/README.md) | 📝 |

### O — Dynamic Routing

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-dynamic-routing-and-ospf-concepts`](O-dynamic-routing/01-dynamic-routing-and-ospf-concepts/README.md) | 📝 |
| 02 | [`02-single-area-ospf`](O-dynamic-routing/02-single-area-ospf/README.md) | 📝 |
| 03 | [`03-ospf-areas-and-default-route`](O-dynamic-routing/03-ospf-areas-and-default-route/README.md) | 📝 |
| 04 | [`04-bgp-basics`](O-dynamic-routing/04-bgp-basics/README.md) | 📝 |

### P — IPv6 Networking

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-ipv6-addressing`](P-ipv6-networking/01-ipv6-addressing/README.md) | 📝 |
| 02 | [`02-neighbor-discovery-and-slaac`](P-ipv6-networking/02-neighbor-discovery-and-slaac/README.md) | 📝 |
| 03 | [`03-dhcpv6-prefix-delegation`](P-ipv6-networking/03-dhcpv6-prefix-delegation/README.md) | 📝 |
| 04 | [`04-ipv6-firewall`](P-ipv6-networking/04-ipv6-firewall/README.md) | 📝 |

### Q — Network Monitoring & Troubleshooting

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-logs-and-remote-syslog`](Q-network-monitoring-and-troubleshooting/01-logs-and-remote-syslog/README.md) | 📝 |
| 02 | [`02-resources-profiler-and-graphing`](Q-network-monitoring-and-troubleshooting/02-resources-profiler-and-graphing/README.md) | 📝 |
| 03 | [`03-snmp-monitoring`](Q-network-monitoring-and-troubleshooting/03-snmp-monitoring/README.md) | 📝 |
| 04 | [`04-traffic-flow`](Q-network-monitoring-and-troubleshooting/04-traffic-flow/README.md) | 📝 |
| 05 | [`05-packet-sniffer-and-wireshark`](Q-network-monitoring-and-troubleshooting/05-packet-sniffer-and-wireshark/README.md) | 📝 |
| 06 | [`06-bandwidth-test`](Q-network-monitoring-and-troubleshooting/06-bandwidth-test/README.md) | 📝 |

## R — Capstone Projects

Projects that combine several chapters into one complete network. For now each folder holds the project brief (🗂️); the full build is added after the topics it uses have their lab evidence.

| # | Folder | Status |
| :-- | :--- | :---: |
| 01 | [`01-complete-home-network`](R-capstone-projects/01-complete-home-network/README.md) | 🗂️ |
| 02 | [`02-small-office-network`](R-capstone-projects/02-small-office-network/README.md) | 🗂️ |
| 03 | [`03-community-wifi-hotspot`](R-capstone-projects/03-community-wifi-hotspot/README.md) | 🗂️ |
| 04 | [`04-dual-isp-office`](R-capstone-projects/04-dual-isp-office/README.md) | 🗂️ |
| 05 | [`05-two-sites-with-dynamic-routing`](R-capstone-projects/05-two-sites-with-dynamic-routing/README.md) | 🗂️ |
