# Nestova Federal Contracting AI Agent  Skill Suite

**Nestova LLC** | UEI: N47XNMUA9369 | CAGE: 21GW7 | NAICS: 423430
email: william@nestova.net | web: supply.nestova.net

---

## Overview

This repository contains all **8 production skills** for the Nestova Federal Contracting AI Agent  an autonomous system that discovers federal procurement opportunities on SAM.gov, scores them for bid-fit, benchmarks pricing against historical awards, sources products, generates complete SF-1449-formatted bid packages, and delivers them for human review and submission.

The agent runs fully autonomously on a daily cron schedule (06:00 AM ET) with no human intervention required between steps.

---

## Pipeline Execution Order

| # | Skill | Trigger | Output |
|---|-------|---------|--------|
| 1 | [Opportunity Discovery](skills/01_opportunity_discovery.md) | Scheduled Cron 06:00 ET | Enriched opportunity array |
| 2 | [Bid-Fit Scoring](skills/02_bid_fit_scoring.md) | Called by Skill 1 | Scored & filtered opportunities |
| 3 | [Competitive Price Benchmarking](skills/03_competitive_benchmarking.md) | Called by Skill 2 | Benchmark-enriched opportunities |
| 4 | [Product Sourcing & Pricing](skills/04_product_sourcing_pricing.md) | Called by Skill 3 | Line-item priced opportunities |
| 5 | [Bid Package Generation](skills/05_bid_package_generation.md) | Called by Skill 4 | Complete bid package documents |
| 6 | [Compliance Self-Check](skills/06_compliance_self_check.md) | Called by Skill 5 | Compliance-verified packages |
| 7 | [Notification & Output](skills/07_notification_output.md) | Called by Skill 6 | Email delivery to operator |
| 8 | [Audit Trail & State Management](skills/08_audit_trail_state.md) | Always-On | Persistent logs & memory |

---

## Required Secrets

| Secret Key | Description | Source |
|------------|-------------|--------|
| SAM_API_KEY | SAM.gov Opportunities API key | api.sam.gov |
| SENDGRID_API_KEY | SendGrid email delivery | sendgrid.com |
| NOTIFICATION_EMAIL | Operator email for bid delivery | william@nestova.net |

---

## Company Identity

```
Company:  Nestova LLC
UEI:      N47XNMUA9369
CAGE:     21GW7
NAICS:    423430
Address:  1152 Rock Springs Rd, Blacksburg, SC
Email:    william@nestova.net
Website:  supply.nestova.net
Owner:    William Jackson, Managing Member
```

---

## Schedule

- **Trigger:** Daily at 06:00 AM Eastern Time
- **Timezone:** America/New_York
- **Cap:** 20 opportunities per run

---

## Tech Stack

| Component | Choice |
|-----------|--------|
| Language | Python 3.12 |
| AI Model | GPT-4o |
| Database | PostgreSQL 16 (AWS RDS) |
| Queue | Celery + Redis |
| Cloud | AWS us-east-1 |
| Security | AES-256, TLS 1.3, RBAC, MFA |
| Compliance | FAR 52.204-21, NIST SP 800-171 |

---

*Version 1.0  July 2026 | Nestova LLC  Internal Engineering Use*
