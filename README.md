# Vector Five9

> Serverless vulnerability management platform built on AWS — purpose-built for network security operations.

![Vector Five9 Dashboard](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF91.jpg)

![CVE Detail with MITRE ATT&CK](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF92.jpg)

![Device Inventory](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF93.jpg)

![Network Scan](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF94.jpg)

![Integrations](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF95.jpg)

![ServiceNow Tickets](https://raw.githubusercontent.com/theonedre/vector-five9-platform/main/VF96.jpg)

---

## Overview

Vector Five9 is a production-grade, fully serverless cybersecurity platform that continuously monitors the [NIST National Vulnerability Database](https://nvd.nist.gov) for CVEs matching a managed network device inventory. When a match is found, the platform scores it using CVSS v3, maps it to [MITRE ATT&CK](https://attack.mitre.org) adversary techniques, and automatically creates a prioritized incident in ServiceNow — all without human intervention.

Built entirely on AWS with zero EC2 instances. Every component is event-driven and scales to zero when idle.

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
2. **CWE → ATT&CK mapping** — NVD weakness classifications mapped to adversary techniques (covers ~80% of CVEs)

Example enrichment for a CVSS 10.0 Cisco CVE:

```
CVE-2026-20182  →  T1078  Valid Accounts      (Defense Evasion)
                   T1110  Brute Force         (Credential Access)
```

Technique cards in the dashboard link directly to detection guidance, data sources, and mitigations on attack.mitre.org — giving the responding engineer actionable intelligence alongside the patch advisory.

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

Fully serverless — cost scales with usage and is near-zero when idle between hourly polls.

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

Built by **Andrei Taylor** — Network Engineer at New Era Networks.

This project was built to solve a real operational problem: manually tracking CVEs across a heterogeneous network device inventory is slow, inconsistent, and error-prone. Vector Five9 automates the entire pipeline — from CVE publication to ServiceNow incident creation — in under 60 seconds, with full MITRE ATT&CK context included in every ticket.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Andrei%20Taylor-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/andrei-taylor/)

---

*This repository contains the project showcase only. Source code is maintained in a private repository.*
