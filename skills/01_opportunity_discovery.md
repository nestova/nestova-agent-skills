# Skill 01 — Opportunity Discovery

**Category:** Data Ingestion
**Trigger:** Scheduled Cron — Daily 06:00 AM Eastern Time
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Poll the SAM.gov Opportunities API every morning for active federal solicitations matching Nestova's NAICS 423430 profile. Filter, deduplicate, and AI-enrich each opportunity before passing to the scoring pipeline.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| run_date | ISO 8601 string | Yes | Current date | Date of execution |
| naics_filter | string | Yes | 423430 | NAICS code to filter on |
| lookback_hours | integer | No | 24 | Hours to look back |
| min_value | number | No | 2500 | Minimum solicitation value ($) |
| max_value | number | No | 5000000 | Maximum solicitation value ($) |
| deadline_buffer_hours | integer | No | 72 | Exclude opps with deadline within this many hours |

---

## API Call

```
Endpoint: GET https://api.sam.gov/prod/opportunities/v2/search
Auth: Header X-Api-Key: {SAM_API_KEY}
Params: naicsCode=423430, limit=100, active=true, responseFormat=json
```

---

## Filter Chain

| Filter | Rule | Rejection Reason |
|--------|------|------------------|
| A | Deadline < now + 72h | deadline_too_soon |
| B | Value < $2,500 | below_minimum_value |
| C | Value > $5,000,000 | above_maximum_value |
| D | noticeId in seen_notice_ids | duplicate |
| E | naicsCode not 423430 | naics_mismatch |

---

## AI Enrichment Prompt

```
You are a federal procurement data enrichment assistant. Given the following
SAM.gov opportunity data, extract and infer these fields. Return JSON only.

Fields:
- product_category: specific IT product category needed
- complexity: simple | moderate | complex
- set_aside_eligible: true | false
- estimated_line_items: integer
- urgency: low | medium | high
- notes: notable requirements or red flags (max 2 sentences)

Opportunity Data:
Title: {title}
Agency: {agency}
Description: {description}
Set-Aside: {set_aside}
Value: {value}
Deadline: {deadline}
PSC Code: {psc_code}
```

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| opportunities | array | Enriched objects passing all filters |
| total_discovered | integer | Raw SAM.gov count before filtering |
| total_passed | integer | Count advancing to Skill 02 |
| total_rejected | integer | Count removed by filters |
| rejection_breakdown | object | Count per rejection reason |
| no_opportunities_today | boolean | True if zero passed |
| run_timestamp | ISO 8601 | Completion timestamp |

---

## Error Handling

| Error | Behavior | Retry |
|-------|----------|-------|
| API 401 | Key invalid | Alert william@nestova.net immediately, no retry |
| API 429 | Rate limit | Wait 60s, exponential backoff, max 3 attempts |
| API 500/503 | Server error | Wait 5 min, retry max 3x, then skip run |
| 0 results | No new postings | Set no_opportunities_today=true, notify, exit |

---

## Configuration

| Key | Value | Required |
|-----|-------|----------|
| SAM_API_KEY | Secret: SAM_API_KEY | Yes |
| NAICS_CODE | 423430 | Yes |
| DAILY_CAP | 20 | No |
| MIN_VALUE | 2500 | No |
| MAX_VALUE | 5000000 | No |
| DEADLINE_BUFFER_HOURS | 72 | No |

---

## Standalone Fleet Prompt

```
SKILL: OPPORTUNITY DISCOVERY
Call SAM.gov /v2/search (naicsCode=423430, active=true, last 24h). Apply
filter chain: deadline >=72h, value $2.5K-$5M, no duplicates, NAICS match.
AI-enrich each passing opp with product_category, complexity,
set_aside_eligible, estimated_line_items, urgency, notes. Update memory
seen_notice_ids. Return enriched array + run stats. If API fails retry 3x
then alert. If zero pass set no_opportunities_today=true and exit.
```

---

*Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
