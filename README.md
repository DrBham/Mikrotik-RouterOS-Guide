<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F766E,50:2563EB,100:7C3AED&height=210&section=header&text=RouterOS%20Field%20Manual&fontSize=38&fontColor=FFFFFF&fontAlignY=38&desc=A%20Practical%20MikroTik%20Networking%20Guide&descAlignY=60&descSize=17" alt="RouterOS Field Manual — A Practical MikroTik Networking Guide" width="100%" />

# RouterOS Field Manual

**A personal learning journal and practical configuration reference for MikroTik RouterOS.**

![MikroTik](https://img.shields.io/badge/MikroTik-RouterOS-293239?style=flat-square&logo=mikrotik&logoColor=white)
![Learning](https://img.shields.io/badge/Learning-Beginner%20to%20Advanced-2563EB?style=flat-square)
![Documentation](https://img.shields.io/badge/Documentation-English-0D9488?style=flat-square)
![Status](https://img.shields.io/badge/Status-Getting%20Started-F59E0B?style=flat-square)

[About](#about-this-project) · [Roadmap](#learning-roadmap) · [Tutorials](#tutorial-format) · [Lab](#lab-environment) · [Progress](#learning-progress)

</div>

## About This Project

I created this repository as a personal learning journal to deepen my understanding of MikroTik RouterOS and build a reliable reference for revisiting networking concepts, configuration procedures, and troubleshooting techniques. My goal is to document what I learn through hands-on practice so that I can recall not only the commands, but also the reasoning behind them.

As the project grows, I hope these notes and practical guides will also help other learners explore MikroTik networking, follow reproducible configuration examples, and understand how to verify their own results.

<table>
<tr>
<td width="50%" valign="top">
<h3>📘 Purpose</h3>
<p>Learn MikroTik from the fundamentals, retain knowledge, and build a personal technical reference.</p>
</td>
<td width="50%" valign="top">
<h3>🧪 Approach</h3>
<p>Practice in physical and virtual labs, record actual results, and document the lessons learned.</p>
</td>
</tr>
<tr>
<td valign="top">
<h3>🎯 Audience</h3>
<p>Networking students, MikroTik learners, MTCNA candidates, IT support technicians, and junior network engineers.</p>
</td>
<td valign="top">
<h3>🌱 Long-Term Goal</h3>
<p>Develop practical networking skills and share a growing, accessible knowledge base with the community.</p>
</td>
</tr>
</table>

## Learning Roadmap

The roadmap contains 17 chapters, arranged from foundational concepts to more advanced networking topics. The categories below are a learning guide, not an official certification syllabus. Some advanced topics appear in earlier chapters and will be approached according to their prerequisites.

<table>
<tr>
<th width="33%">🟢 Beginner</th>
<th width="34%">🔵 Intermediate</th>
<th width="33%">🟣 Advanced</th>
</tr>
<tr>
<td valign="top">
<b>01</b> — MikroTik Fundamentals<br>
<b>02</b> — Network Fundamentals &amp; IP Addressing<br>
<b>03</b> — Basic Internet Connectivity<br>
<b>04</b> — Interface &amp; Bridge<br>
<b>05</b> — DHCP &amp; Network Services<br>
<b>06</b> — Wireless Networking<br>
<b>07</b> — Routing Fundamentals
</td>
<td valign="top">
<b>08</b> — Firewall &amp; Network Security<br>
<b>09</b> — Network Address Translation (NAT)<br>
<b>10</b> — VLAN &amp; Network Segmentation<br>
<b>11</b> — Bandwidth Management &amp; QoS<br>
<b>12</b> — PPP &amp; Hotspot<br>
<b>13</b> — VPN &amp; Network Tunneling
</td>
<td valign="top">
<b>14</b> — Load Balancing &amp; High Availability<br>
<b>15</b> — Dynamic Routing<br>
<b>16</b> — IPv6 Networking<br>
<b>17</b> — Network Monitoring &amp; Troubleshooting
</td>
</tr>
<tr>
<td valign="top"><i>Build and troubleshoot basic networks.</i></td>
<td valign="top"><i>Implement segmentation, security, and network services.</i></td>
<td valign="top"><i>Explore redundancy, dynamic routing, and network operations.</i></td>
</tr>
</table>

> **Roadmap note:** Chapters and topics represent planned coverage. Tutorials will be added as the author studies, implements, and reviews them. Chapter folders use letters (01 = `A-…`, 02 = `B-…`, and so on). The topic-by-topic plan is in [ROADMAP.md](ROADMAP.md).

## Tutorial Format

Each topic has its own folder. The guide is written for **beginners** who are new to networking, and for **experienced readers** who want to refresh theory and commands quickly. Every topic therefore explains *why* it matters and *how it connects* to other topics, and ends with a short **Quick Recap**.

Practical tutorials show the same configuration twice. Both walkthroughs reach the same result, and both are documented with screenshots from the lab, so readers can follow whichever tool they prefer.

<table>
<tr>
<td width="50%" valign="top">
<h3>🖥️ Method A — WinBox (GUI)</h3>
Step-by-step graphical configuration with original WinBox screenshots for each step.
</td>
<td width="50%" valign="top">
<h3>⌨️ Method B — RouterOS CLI</h3>
The equivalent terminal commands, an explanation of each parameter, and a screenshot of the command and its output.
</td>
</tr>
<tr>
<td valign="top">
<h3>✅ Verification &amp; Troubleshooting</h3>
Expected behavior, observed results, common issues, diagnostic steps, and recovery guidance where relevant.
</td>
<td valign="top">
<h3>📄 Configuration File</h3>
A topic-specific <code>configuration.rsc</code> file, reviewed for sensitive information and accompanied by prerequisites and usage notes.
</td>
</tr>
</table>

Apply only one method on your router; applying both creates duplicate entries.

**Learning workflow:** Understand → Plan → Implement → Verify → Document → Publish

Theory topics contain explanations and illustrations without configuration files. See [TOPIC-TEMPLATE.md](TOPIC-TEMPLATE.md) for the full section layout and [ROADMAP.md](ROADMAP.md) for every planned topic.

## Sources

Theory is based on the official [RouterOS Manual](https://manual.mikrotik.com/), MikroTik product pages, and IETF RFCs. Each statement in a topic is numbered against that topic's **References** list. My own lab observations are labelled as such and are not presented as general facts.

## Repository Structure

The following example illustrates the planned organization. Topic folders are created incrementally as learning progresses.

```text
Mikrotik-RouterOS-Guide/
├── README.md
├── ROADMAP.md
├── TOPIC-TEMPLATE.md
├── A-mikrotik-fundamentals/
│   ├── 01-introduction-to-mikrotik/
│   │   └── README.md
│   └── 02-connecting-to-routeros-using-winbox/
│       ├── README.md
│       └── images/
│           ├── topology.svg
│           ├── gui/
│           └── cli/
└── B-network-fundamentals/
    └── 11-static-ip-configuration/
        ├── README.md
        ├── images/
        │   ├── gui/01-open-ip-addresses.png
        │   └── cli/01-ip-address-add.png
        └── configuration.rsc
```

| Component | Purpose |
| :--- | :--- |
| `README.md` | Concept, configuration steps, verification, and troubleshooting. |
| `images/gui/`, `images/cli/` | Original lab screenshots for the WinBox and CLI walkthroughs. |
| `configuration.rsc` | Relevant RouterOS configuration for the practical topic. |

## Lab Environment

<table>
<tr>
<th width="50%">🔧 Physical Lab</th>
<th width="50%">☁️ Virtual Lab</th>
</tr>
<tr>
<td valign="top">MikroTik hAP lite (RB941-2nD)<br>Lenovo LOQ 15IAX9<br>Ethernet connectivity</td>
<td valign="top">MikroTik Cloud Hosted Router (CHR)<br>GNS3<br>Oracle VirtualBox</td>
</tr>
<tr>
<th>🛠️ Configuration &amp; Analysis</th>
<th>📝 Documentation &amp; Version Control</th>
</tr>
<tr>
<td valign="top">WinBox<br>RouterOS CLI<br>Wireshark (when needed)</td>
<td valign="top">Visual Studio Code<br>Markdown<br>Git &amp; GitHub</td>
</tr>
</table>

Each practical tutorial records its actual RouterOS version, device, topology, and test environment. Hardware- or version-dependent features are documented accordingly.

## Learning Progress

| Milestone | Status |
| :--- | :--- |
| Repository initialization | ✅ Completed |
| Detailed roadmap and topic template | ✅ Completed |
| A.01 — Introduction to MikroTik | 📝 Draft |
| A.02 — Connecting to RouterOS Using WinBox | 📝 Draft, waiting for lab screenshots |
| Chapter B — Network Fundamentals | ⏳ Planned |
| Chapter C — Basic Internet Connectivity | ⏳ Planned |

## Documentation & Safety

Practical guides are based on actual lab evidence and original explanations. Untested configurations are explicitly identified. Review configuration examples before applying them, use isolated labs when possible, and avoid publishing credentials, private keys, or identifying device information. Published examples are not a guarantee of production suitability.

## References

| Resource | Description |
| :--- | :--- |
| [RouterOS Manual](https://manual.mikrotik.com/) | Primary technical reference (current official manual). |
| [RFC Editor](https://www.rfc-editor.org/) | Protocol standards used for general networking theory. |
| [MikroTik Official Website](https://mikrotik.com/) | Product information and official resources. |

---

<div align="center">

**Learn it. Configure it. Understand it. Document it.**

<sub>An independent personal learning project. Not affiliated with MikroTik and not official certification training material.</sub>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F766E,50:2563EB,100:7C3AED&height=90&section=footer" alt="Decorative footer" width="100%" />

</div>
