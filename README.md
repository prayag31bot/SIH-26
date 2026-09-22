# ACTGuard: AI-Driven Multi-Vendor Network Security Compliance Auditor

> One AI auditor for every network vendor.

**Smart India Hackathon 2026** | Problem Statement **SIH26155** | Theme: Blockchain & Cybersecurity | Team **Runtime Terror**

[Demo Video](#demo-video) · [Presentation](https://docs.google.com/presentation/d/1JxQUsyW2gYUP166sUOouct6gQdTw0NVH06u1ckBVdOc/edit?usp=sharing)

## Demo Video

[![ACTGuard Demo Video](https://www.youtube.com/watch?si=JRNqOl5N6BS68iZK&v=Z6p6MkNDxqs&feature=youtu.be)

**Watch on YouTube:** https://www.youtube.com/watch?si=JRNqOl5N6BS68iZK&v=Z6p6MkNDxqs&feature=youtu.be

## Table of Contents
1. [Overview](#overview)
2. [Key Features](#key-features)
3. [How It Works](#how-it-works)
4. [Architecture](#architecture)
5. [Tech Stack](#tech-stack)
6. [Usage](#usage)
7. [Adding a New Vendor (Training Loop)](#adding-a-new-vendor-training-loop)
8. [Compliance Rules](#compliance-rules)
9. [Security and Privacy](#security-and-privacy)
10. [Team](#team)
11. [References](#references)


## Overview

Enterprise networks mix devices from many vendors (Cisco, Juniper, Palo Alto, Fortinet and more), and each uses its own CLI syntax. Misconfiguration is a leading cause of firewall breaches, yet audits are still done by manual checklists or by costly, vendor-locked suites.

**ACTGuard** reads raw configuration files from any vendor, converts them into a vendor-neutral **Abstract Configuration Tree (ACT)**, and checks that tree against security frameworks. For every device it produces a PDF report with Pass/Fail results, severity, and device-specific CLI commands to fix each issue.

When it meets a configuration structure it does not recognise, an **interactive training GUI** lets the administrator tag the unknown commands. The parser learns from that input without any code redeployment.

## Key Features

- **Unified ingestion:** upload a single config or a ZIP of many, from any device.
- **Vendor-neutral ACT:** one Security Baseline Model for all vendors.
- **Multi-framework engine:** CIS Benchmarks, NIST SP 800-53, DISA STIGs and ISO/IEC 27001.
- **Training loop:** an admin-assisted GUI to teach the system new vendors and syntax.
- **Per-device PDF report:** device identification, Pass/Fail with severity, and step-by-step CLI remediation.
- **Bulk fleet mode:** parallel audits with a fleet dashboard and cross-device checks (Neo4j).
- **Tamper-evident reports:** each report is SHA-256 hashed and chained to the previous record.
- **Offline-first:** runs in Docker with no external API calls by default.

## How It Works

| Mode | Use it when | What happens |
|---|---|---|
| **1. Known vendor** | The vendor is already supported | Upload → detect OS → parse to ACT → select framework → audit → PDF |
| **2. Training loop** | Lines are not recognised | Low-confidence lines flagged → admin tags them in the GUI → parser updated → saved as a versioned vendor profile |
| **3. Bulk fleet** | Many devices at once | Upload ZIP → parallel audit → cross-device checks → dashboard → PDF per device and a sealed manifest |

**Design principle:** AI only *proposes* a category for unknown lines. A deterministic rule engine makes every Pass/Fail decision, and the admin approves every learned mapping.

## Architecture

```
Admin login (RBAC)
      │
Unified Ingestion (single / bulk)
      │
Vendor recognised? ──No──► AI tagger + Admin Training GUI
      │Yes                          │
Vendor parser (Pyparsing) ◄─────────┘
      │
ACT (Security Baseline Model)
      │
Compliance Engine (YAML rules → framework controls)
      │
Findings: Pass/Fail · Severity
      │
Device PDF + CLI fixes + SHA-256 seal
```


## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python, FastAPI |
| Parsing | Pyparsing, regex |
| AI / NLP | spaCy, local model for tagging unseen lines |
| Frontend | React, Tailwind CSS |
| Graph | Neo4j |
| Storage | PostgreSQL, YAML rule files |
| Reports | ReportLab, SHA-256 |
| Deployment | Docker, Docker Compose |


## Usage

1. Log in to the web app.
2. **Upload** one config file or a ZIP of several.
3. **Select** the framework(s): CIS, NIST, STIG or ISO 27001.
4. Review the **findings** (Pass/Fail, severity, config line, fix).
5. **Download** the PDF report for each device.


## Adding a New Vendor (Training Loop)

1. Upload a config from the new vendor.
2. Lines the parser cannot classify are flagged in the **Training** page.
3. For each line, choose a security category (for example, "session timeout").
4. Preview the re-parse and confirm.
5. The mapping is saved as a **versioned vendor profile** in `profiles/`. No code change or redeployment is needed.

## Compliance Rules

```yaml
id: CIS-SSH-001
title: Ensure SSH version 2 is used
frameworks:
  cis: "[control id]"
  nist_800_53: "[control id]"
severity: high
check:
  path: management.ssh.version
  operator: equals
  expected: 2
remediation:
  cisco_ios: |
    ip ssh version 2
  fortios: |
    config system global
      set admin-ssh-v1 disable
    end
```

## Security and Privacy

- Configurations are processed **locally**. No external API is called by default.
- Secrets (passwords, keys, community strings) are **masked** before any AI step and never printed in reports.
- Each report is hashed with **SHA-256** and linked to the previous record, so changes are detectable.
- Access is protected by login and role-based access control.

## Team

**Team Runtime Terror**

|---|---|
| Prayag Shah   | [R&D, Documentation] |
| Aswin Rout    | [Team Leader, Backend] |
| Hetvi Shah    | [Presentation] |
| Kush Patel    | [Supporter] |
| Yansi Valand  | [Frontend] |
| Lakshya Dubey | [Backend] |

## References

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-41 Rev. 1](https://csrc.nist.gov/pubs/sp/800/41/r1/final)
- [DISA STIGs](https://public.cyber.mil/stigs/)
- [ISO/IEC 27001](https://www.iso.org/standard/27001)
- [Batfish](https://batfish.org/), prior art in vendor-agnostic config analysis
- LLM agents with a vendor-agnostic intermediate representation for configs: [arXiv 2509.20600](https://arxiv.org/html/2509.20600v1), [arXiv 2501.08760](https://arxiv.org/html/2501.08760v1)
