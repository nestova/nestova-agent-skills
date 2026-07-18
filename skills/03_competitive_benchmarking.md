# Skill 03 — Competitive Price Benchmarking

**Category:** Market Intelligence
**Trigger:** Called by Skill 02
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Query USASpending.gov for historical award data matching each opportunity's NAICS/PSC codes, then use AI to synthesize a competitive price range and win-probability estimates. Ensures Nestova bids are priced to win.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| passed_opportunities | array | Yes | -- | Scored opportunities from Skill 02 |
| lookback_years | integer | No | 3 | Years of award history to query |

---

## USASpending API Call

```
Endpoint: POST https://api.usaspending.gov/api/v2/awards/search/
Auth: None (public API)

Request body:
{
  "filters": {
    "naics_codes": ["423430"],
    "psc_codes": ["{psc_code}"],
    "time_period": [{"start_date": "{3_years_ago}", "end_date": "{today}"}],
    "award_type_codes": ["A","B","C","D"]
  },
  "fields": ["Award Amount","Recipient Name","Award Date","PSC Code"],
  "sort": "Award Amount",
  "order": "desc",
  "limit": 10
}
```

---

## Statistical Calculation

```
benchmark_avg  = arithmetic mean of award_amounts
benchmark_low  = 10th percentile of award_amounts
benchmark_high = 90th percentile of award_amounts
top_competitor = most frequently appearing recipient_name
```

---

## AI Price Intelligence Prompt

```
You are a federal contracting pricing analyst for NAICS 423430.
Analyze the following historical award data and current solicitation
to provide competitive pricing intelligence.

Historical Awards:
{award_history_json}
Benchmark Low: ${benchmark_low}
Benchmark Avg: ${benchmark_avg}
Benchmark High: ${benchmark_high}
Top Winner: {top_competitor}

Current Solicitation:
Title: {title} | Agency: {agency}
Estimated Value: {value}
AI-Inferred Products: {recommended_products}
Set-Aside: {set_aside}

Return ONLY valid JSON:
{
  "benchmark_low": 0.00,
  "benchmark_high": 0.00,
  "benchmark_avg": 0.00,
  "recommended_target_price": 0.00,
  "pricing_strategy": "aggressive|competitive|conservative",
  "win_probability_at_low": "35%",
  "win_probability_at_mid": "62%",
  "win_probability_at_high": "78%",
  "top_competitors": [{"name": "...", "typical_strategy": "..."}],
  "agency_estimate_assessment": "accurate|likely_higher|likely_lower",
  "data_source": "usaspending|ai_estimate",
  "confidence": "HIGH|MEDIUM|LOW",
  "pricing_notes": "2-3 sentence strategic note"
}
```

---

## AI-Only Fallback

Trigger when USASpending returns fewer than 3 records:

```
USASpending returned <3 awards for PSC {psc_code} under NAICS 423430.
Using knowledge of federal IT procurement pricing (2025-2026), estimate
a competitive price range. Set data_source='ai_estimate', confidence='LOW'.
```

---

## PSC Cache

```
Store results -> memory key: psc_benchmark_cache
Key: PSC code | TTL: 7 days
Reuse cached data for duplicate PSC codes in same run.
```

---

## Output Schema

| Field | Type | Description |
|-------|------|-------------|
| benchmarked_opportunities | array | Opportunities with full pricing intelligence |
| benchmarking_summary | object | opportunities_with_data, ai_estimated, avg_range_width |

---

## Error Handling

| Error | Behavior | Retry |
|-------|----------|-------|
| USASpending 429 | Rate limit | Wait 30s, retry |
| PSC not found | Zero results | Trigger AI-only fallback |
| AI price outside 10x range | Validation anomaly | Use estimated_value as fallback |
| Network timeout | Connection failure | Retry 2x, then AI fallback |

---

## Configuration

| Key | Value | Required |
|-----|-------|----------|
| LOOKBACK_YEARS | 3 | No |
| MIN_AWARD_SAMPLES | 3 | No |
| PSC_CACHE_TTL_DAYS | 7 | No |

---

## Standalone Fleet Prompt

```
SKILL: COMPETITIVE PRICE BENCHMARKING
For each passed opportunity, call USASpending POST /v2/awards/search/
(naics=423430, PSC code, last 3 years, types A/B/C/D, limit 10).
Calculate benchmark_low (10th pct), benchmark_avg (mean),
benchmark_high (90th pct), top_competitor (most frequent).
Run AI Price Intelligence prompt for recommended_target_price,
pricing_strategy, win probabilities, competitor analysis.
If <3 USASpending records, use AI-only fallback (confidence=LOW).
Cache by PSC code in psc_benchmark_cache (TTL 7 days).
Return benchmarked_opportunities with full pricing intelligence.
```

---

*Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
