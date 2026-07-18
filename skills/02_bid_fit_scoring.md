# Skill 02 — Bid-Fit Scoring

**Category:** AI Reasoning
**Trigger:** Called by Skill 01
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Evaluate each discovered opportunity against Nestova's NAICS 423430 capability profile using structured AI reasoning. Assign a bid-fit score from 0.00 to 1.00. Only opportunities scoring >= 0.75 advance through the pipeline.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| opportunities | array | Yes | -- | Enriched opportunity objects from Skill 01 |
| scoring_threshold | float | No | 0.75 | Minimum score required to advance |
| batch_size | integer | No | 5 | Opportunities per AI scoring batch |

---

## AI Scoring Prompt

```
You are a federal contracting bid analyst for Nestova LLC, a registered
small business IT product reseller and merchant wholesaler.

Company Profile:
- Legal Name: Nestova LLC
- UEI: N47XNMUA9369 | CAGE: 21GW7
- Primary NAICS: 423430 (Computer and Computer Peripheral Equipment and
  Supplies, Merchant Wholesalers)
- Business Type: Small Business
- Capabilities: IT hardware supply, desktop and laptop computers, computer
  peripherals, office electronics, printer supplies and consumables,
  networking equipment and accessories, cables and connectors, monitors
  and displays, tablets, input devices, storage devices and media
- NOT capable of: Professional IT services, software development, staffing,
  construction, healthcare products, food service, janitorial supplies

Scoring Criteria:

HIGH score (0.75-1.00) when ALL true:
  Physical IT products sourceable from commercial distributors
  Compatible set-aside: SB, 8(a), HUBZone, SDVOSB, or Unrestricted
  Value between $2,500 and $5,000,000
  Delivery >= 7 business days ARO
  No security clearance required for the products
  Commercially available via CDW-G, Ingram Micro, TD SYNNEX, SHI, Zones

LOW score (0.00-0.49) when ANY true:
  Services-only requirement
  Custom manufacturing or fabrication required
  Set-aside explicitly excludes Nestova
  Complex multi-item delivery < 5 business days
  On-site installation is primary deliverable
  ITAR/EAR controlled or military-spec products

MEDIUM score (0.50-0.74): partial matches or uncertain scope

Return ONLY valid JSON:
{
  "notice_id": "{notice_id}",
  "score": 0.00,
  "pass": false,
  "confidence": "HIGH|MEDIUM|LOW",
  "product_match": "exact|partial|none",
  "set_aside_eligible": true,
  "primary_concern": "reason if score < 0.75, else null",
  "rationale": "2-3 sentence explanation",
  "recommended_products": ["product type 1", "product type 2"],
  "estimated_margin_band": "15-22%",
  "risk_flags": ["flag 1", "flag 2"]
}

Opportunity:
Notice ID: {notice_id}
Title: {title}
Agency: {agency}
Description: {description}
PSC Code: {psc_code}
Set-Aside: {set_aside}
Value: {value}
Deadline: {deadline}
Product Category: {product_category}
```

---

## Threshold Filter

```
score >= 0.75 -> status: passed_scoring  -> advance to Skill 03
score <  0.75 -> status: rejected_scoring -> log and stop
Log ALL scores regardless of outcome.
```

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| passed_opportunities | array | Opportunities scoring >= threshold |
| rejected_opportunities | array | Opportunities below threshold with rationale |
| scoring_summary | object | total_scored, total_passed, total_rejected, avg_score |

---

## Error Handling

| Error | Behavior | Retry |
|-------|----------|-------|
| AI returns malformed JSON | Re-prompt with JSON-only reminder | Retry once per opportunity |
| AI scores all at 0.00 | Data issue suspected | Flag batch for human review, continue |
| Batch times out | Timeout | Reduce batch to 2, retry failed batch only |

---

## Configuration

| Key | Value | Required |
|-----|-------|----------|
| SCORING_THRESHOLD | 0.75 | No |
| BATCH_SIZE | 5 | No |
| MAX_RETRIES | 2 | No |
| PRIORITY_SET_ASIDES | SBA,VSB,8A | No |

---

## Standalone Fleet Prompt

```
SKILL: BID-FIT SCORING
Score each opportunity 0.00-1.00 against Nestova LLC NAICS 423430 profile.
HIGH (>=0.75): physical IT products, compatible set-aside, $2.5K-$5M value,
>=7 day delivery, commercially sourceable via CDW-G/Ingram/TD SYNNEX.
LOW (<0.50): services-only, custom mfg, incompatible set-aside.
Return JSON: score, pass, confidence, rationale, recommended_products, risk_flags.
Apply threshold 0.75. Sort passed by score desc. Log all, update memory.
```

---

*Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
