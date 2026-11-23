# Topic Template

Copy this structure into a new topic folder's `README.md`. Delete sections marked *(Practice only)* for theory topics.

The guide is written for two kinds of readers:

- **Beginners** read from top to bottom: why, theory, then practice.
- **Experienced readers** jump to **Quick Recap** for a refresher of the theory and commands.

## Folder layout

```text
NN-topic-name/
├── README.md
├── images/
│   ├── topology.svg            # lab diagram (if needed)
│   ├── gui/                    # Method A — WinBox screenshots
│   │   ├── 01-open-ip-addresses.png
│   │   └── 02-add-ip-address.png
│   └── cli/                    # Method B — terminal screenshots
│       ├── 01-ip-address-add.png
│       └── 02-ip-address-print.png
└── configuration.rsc           # (Practice only) final config, no secrets
```

Screenshot names start with the step number they belong to, so `gui/03-…` and `cli/03-…` show the same step done two ways.

## README sections

````markdown
<div align="center">

# Topic Title

*Chapter X — Chapter Name | Topic X.NN*

![Level](https://img.shields.io/badge/Level-Beginner-16A34A) ![Type](https://img.shields.io/badge/Type-Practice-7C3AED)

</div>

## Overview
What this topic covers, in two or three sentences, and what the reader can do afterwards.

## Why This Matters
The real problem this topic solves. Example: "A router without a default route
cannot reach any network it is not directly connected to." Cite the source.

## Prerequisites
| Topic | Why it is needed |
| --- | --- |
| [B.04 — IPv4 Addresses](../../B-network-fundamentals/04-…/README.md) | … |

## Theory
Concepts needed for this topic. Every factual statement ends with a reference
number, e.g. "...on TCP port 8291. [2]". My own observations are labelled
**Lab note:**, not cited as general fact.

## How It Connects
How this topic relates to earlier and later topics (one short paragraph or a diagram).

## Lab Environment            *(Practice only)*
| Item | Value |
| --- | --- |
| Device | MikroTik hAP lite (RB941-2nD) |
| RouterOS version | 7.xx (from `/system/resource/print`) |
| WinBox version | 4.x |
| Topology | ![Topology](images/topology.svg) |

## Method A — WinBox (GUI)    *(Practice only)*
### Step 1 — …
What to click and why.
![Step 1](images/gui/01-….png)

## Method B — CLI            *(Practice only)*
### Step 1 — …
```routeros
/ip/address/add address=192.168.88.1/24 interface=ether2
```
What each parameter means.
![Step 1](images/cli/01-….png)

## Verification              *(Practice only)*
Expected result, then the actual result from the lab (screenshot).

## Troubleshooting           *(Practice only)*
| Symptom | Likely cause | Check / fix |
| --- | --- | --- |

## Quick Recap
| Concept / Command | Meaning |
| --- | --- |
Short refresher for readers who already know the topic.

## Key Takeaways
- …

## References
1. [RouterOS Manual — Page title](https://manual.mikrotik.com/docs/…)

---
**Previous:** … · **Next:** …
````
