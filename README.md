# Vector Five9

> Serverless vulnerability management platform built on AWS purpose-built for network security operations.

![Vector Five9 Dashboard](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF91.jpg)

![CVE Detail with MITRE ATT&CK](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF92.jpg)

![Device Inventory](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF93.jpg)

![Network Scan](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF94.jpg)

![Integrations](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF95.jpg)

![ServiceNow Tickets](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF96.jpg)

---

## Overview

Vector Five9 is a production-grade, fully serverless cybersecurity platform that continuously monitors the [NIST National Vulnerability Database](https://nvd.nist.gov) for CVEs matching a managed network device inventory. When a match is found, the platform scores it using CVSS v3, maps it to [MITRE ATT&CK](https://attack.mitre.org) adversary techniques, and automatically creates a prioritized incident in ServiceNow all without human intervention.

Built entirely on AWS with zero EC2 instances. Every component is event-driven and scales to zero when idle.


## About Me

## Andrei Taylor

Before anything else, I want to make one thing clear: I am not a software developer or software engineer. I have tremendous respect for the professionals in this field, and this project has given me an even greater appreciation for the skill, dedication, patience, and expertise required to build meaningful software solutions. I would never compare my journey to the years of discipline and experience it takes to become proficient in software development.

What I am is someone who is naturally curious, passionate about technology, and unafraid to tackle difficult challenges head-on.

Over the course of my career, I have had the opportunity to work for several large organizations in roles including Network Manager, Network Engineer, and most recently, Data Center Operations. These positions have provided me with firsthand experience supporting critical infrastructure, troubleshooting complex environments, and ensuring operational continuity for businesses that rely on technology every day.

While my primary responsibilities have centered around networking and infrastructure, I have consistently been involved in cybersecurity-related initiatives and tasks. This exposure has given me a unique perspective on the daily challenges faced by security teams. I have seen the pressure of responding to threats in real time, the overwhelming volume of alerts, and the constant need to balance security, efficiency, and limited resources.

This application was born from those experiences.

Throughout the years, I have repeatedly encountered situations where automation could have significantly increased productivity, improved visibility, and reduced the time required to identify and respond to critical issues. In mission-critical environments, time is one of the most valuable commodities. Every second spent searching through data, correlating information, or manually performing repetitive tasks is time that could be spent addressing the actual threat.

As cyber threats continue to evolve and become more sophisticated, the demands placed on security professionals continue to grow. My passion for cybersecurity, combined with my eagerness to learn and explore the capabilities of artificial intelligence, inspired me to create a solution that helps address some of these challenges.

The purpose of this application is not to replace existing security tools, platforms, or methodologies. Instead, it is designed to complement and enhance them. The goal is to provide purpose-driven automation that increases visibility into high-priority security concerns, helping organizations and individuals identify what matters most and respond more efficiently.

I am particularly passionate about creating solutions that lower barriers to entry. Not every organization has access to large security teams or expensive enterprise platforms. By leveraging modern technologies such as AI, I believe it is possible to build practical tools that empower defenders, improve operational effectiveness, and strengthen security posture without requiring massive investments.

Seeing this vision come to life has been incredibly rewarding. While this is only the beginning, I am excited about future enhancements, integrations, and capabilities that will continue to improve the platform and expand its value.

At its core, this project represents a simple philosophy: solve real-world problems efficiently, increase visibility where it matters most, and make powerful technology more accessible to the people who need it.

Technology should be practical, accessible, and empowering, this application is my contribution towards that goal.


---

## Key Capabilities

| Capability | Detail |
|---|---|
| **CVE Monitoring** | Polls NIST NVD every hour via EventBridge + Lambda |
| **Device Matching** | CPE keyword matching against managed device inventory |
| **CVSS Scoring** | v3.1 base scores with severity banding (Critical / High / Medium / Low) |
| **MITRE ATT&CK** | CWE → ATT&CK correlation maps each CVE to adversary techniques |
| **Auto-Ticketing** | ServiceNow incidents auto-created with SLA-mapped priority |
| **Firmware Scanning** | SNMP / SSH / HTTP / Nmap probing for active firmware fingerprinting |
| **Custom CVE Scans** | Date-range scans with per-device filtering from the dashboard |
| **Alert Routing** | CRITICAL/HIGH → P1/P2 tickets + SNS email · MEDIUM → Security Team · LOW → log only |

---

## Architecture

```
                    ┌─────────────────────────────────────────────┐
                    │                  AWS Cloud                   │
                    │                                              │
  NIST NVD API ────►│  EventBridge (hourly)                       │
                    │        │                                     │
                    │        ▼                                     │
                    │  NvdPoller Lambda ────► DynamoDB (CVEs)     │
                    │                              │               │
                    │                       DynamoDB Streams       │
                    │                        │          │          │
                    │                        ▼          ▼          │
                    │               CveMatcher    ATT&CK Enricher  │
                    │                   │                          │
                    │             SNS Alert                        │
                    │                   │                          │
                    │           TicketDispatcher ───► ServiceNow   │
                    │                                              │
  Browser ─────────►│  CloudFront → S3 (React Dashboard)          │
                    │        │                                     │
                    │        ▼                                     │
                    │  API Gateway → DeviceApi Lambda              │
                    │                     │                        │
                    │               DynamoDB (Devices)             │
                    └─────────────────────────────────────────────┘
```

**All compute is Lambda. No servers to patch, scale, or maintain.**

---

## Tech Stack

### Cloud Infrastructure
- **AWS Lambda** — 7 serverless functions (Python 3.11)
- **Amazon DynamoDB** — 4 tables with DynamoDB Streams
- **Amazon API Gateway** — REST API, 14 routes
- **Amazon CloudFront** — CDN for React SPA
- **Amazon S3** — Frontend hosting + spreadsheet uploads
- **Amazon EventBridge** — Hourly CVE poll schedule
- **Amazon SNS** — Real-time alert notifications
- **AWS Secrets Manager** — Encrypted credential storage
- **AWS CDK (TypeScript)** — All infrastructure as code

### Application
- **React 18** + TypeScript — Frontend dashboard
- **Python 3.11** — All Lambda business logic
- **Inter** font — UI typography
- **CVSS v3.1** — Vulnerability scoring standard

### Integrations
- **NIST NVD API v2** — Primary CVE data source
- **MITRE ATT&CK** — Adversary technique correlation
- **CISA KEV** — Known exploited vulnerabilities feed
- **ServiceNow REST API** — Incident management and ticketing

---

## Dashboard Features

- **Real-time CVE feed** — sorted by CVSS score, filterable by severity
- **CVE detail modal** — full description, CVSS vector, CWE classification, and MITRE ATT&CK technique cards
- **ATT&CK technique cards** — color-coded by tactic, clickable to attack.mitre.org detection and mitigation guidance
- **Device inventory** — card and table views with per-device CVE count badges
- **Custom date-range scans** — pick any date window and target specific devices
- **Firmware fingerprinting** — active network probing via SNMP, SSH, HTTP, and Nmap
- **ServiceNow ticket management** — view and create incidents directly from the dashboard
- **Dark / Light mode** — persisted per user session
- **Integrations page** — connect Jira, PagerDuty, Slack, Tenable.io, and Qualys

---

## CVSS → ServiceNow Priority Mapping

| CVSS Score | Severity | Priority | SLA | Action |
|---|---|---|---|---|
| 9.0 – 10.0 | CRITICAL | P1 | 4 hours | Auto-ticket + SNS email alert |
| 7.0 – 8.9 | HIGH | P2 | 24 hours | Auto-ticket + SNS email alert |
| 4.0 – 6.9 | MEDIUM | P3 | 7 days | Auto-ticket → Security Team |
| < 4.0 | LOW | P4 | — | Log only |

---

## MITRE ATT&CK Integration

Each CVE match is automatically enriched with MITRE ATT&CK techniques using two methods:

1. **Direct CVE lookup** via CTID Mappings Explorer API
2. **CWE → ATT&CK mapping** NVD weakness classifications mapped to adversary techniques (covers ~80% of CVEs)

Example enrichment for a CVSS 10.0 Cisco CVE:

```
CVE-2026-20182  →  T1078  Valid Accounts      (Defense Evasion)
                   T1110  Brute Force         (Credential Access)
```

Technique cards in the dashboard link directly to detection guidance, data sources, and mitigations on attack.mitre.org, giving the responding engineer actionable intelligence alongside the patch advisory.

---

## Supported Device Types

The platform supports any network device with a CPE identifier in NVD, including:

- Cisco IOS / IOS-XE switches and routers
- TP-Link managed switches and routers (Omada series)
- Palo Alto PAN-OS firewalls
- Fortinet FortiGate
- Juniper JunOS
- Any device reachable via SNMP, SSH, or HTTP management interface

---

## Cost Profile

Fully serverless: cost scales with usage and is near-zero when idle between hourly polls.

| Component | Monthly Estimate |
|---|---|
| NAT Gateway | ~$32 |
| Lambda + API Gateway | ~$2–5 |
| DynamoDB | ~$1–3 |
| CloudFront + S3 | < $1 |
| **Total** | **~$35–40/month** |

---

## Project Status

| Feature | Status |
|---|---|
| Production AWS deployment | ✅ Live |
| React dashboard | ✅ Live |
| Hourly NVD polling | ✅ Active |
| ServiceNow integration | ✅ Active |
| MITRE ATT&CK enrichment | ✅ Active |
| CVE matches detected | ✅ 50+ across 4 devices |
| Dark / Light mode | ✅ Complete |
| Custom CVE date-range scans | ✅ Complete |

---

## About

Built by **Andrei Taylor**, Founder Dynamic Nines LLC.

This project was built to solve a real operational problem: manually tracking CVEs across a heterogeneous network device inventory is slow, inconsistent, and error-prone. Vector Five9 automates the entire pipeline, from CVE publication to ServiceNow incident creation in under 60 seconds, with full MITRE ATT&CK context included in every ticket.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Andrei%20Taylor-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/andrei-taylor/)

---

*This repository contains the project showcase only. Source code is maintained in a private repository.*


