# Skill 08 — Audit Trail and State Management

**Category:** Compliance and Persistence  
**Trigger:** Always-On (called by all skills throughout the pipeline)  
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Maintain a complete, immutable audit trail of every agent action across the full pipeline. Manage persistent state in agent memory so the agent improves over time — tracking seen opportunities, historical pricing, competitor intelligence, and cumulative performance metrics. Every action taken by any skill must produce an audit log entry.

---

## Event Types

| Event Type | Emitted By | Description |
|------------|-----------|-------------|
| `run_started` | Skill 01 | Daily pipeline begins |
| `opportunity_discovered` | Skill 01 | SAM.gov opportunity retrieved |
| `opportunity_filtered` | Skill 01 | Opportunity removed by filter chain |
| `opportunity_scored` | Skill 02 | Bid-fit score assigned |
| `opportunity_rejected_scoring` | Skill 02 | Score below threshold |
| `benchmark_fetched` | Skill 03 | USASpending data retrieved |
| `benchmark_ai_estimated` | Skill 03 | AI fallback used |
| `products_sourced` | Skill 04 | Line-item table generated |
| `pricing_corrected` | Skill 04 | Auto-correction applied |
| `bid_package_generated` | Skill 05 | Full bid document assembled |
| `compliance_checked` | Skill 06 | Compliance review completed |
| `compliance_flagged` | Skill 06 | Package flagged for human review |
| `notification_sent` | Skill 07 | Email delivered to william@nestova.net |
| `run_completed` | Skill 07 | Daily pipeline execution complete |
| `state_updated` | Skill 08 | Agent memory keys updated |
| `api_error` | Any | External API call failed |

---

## Persistent Memory Keys

- `seen_notice_ids` — append-only; all processed SAM.gov notice IDs (deduplication)
- `scored_opportunity_ids` — all AI-scored opportunity IDs
- `psc_benchmark_cache` — PSC pricing data keyed by PSC code, 7-day TTL
- `historical_bid_amounts` — all generated bid totals by PSC code and run date
- `competitor_registry` — USASpending recipient intelligence
- `total_runs_lifetime` / `total_packages_lifetime` / `total_pipeline_value_lifetime` — cumulative counters
- `last_run_date` / `last_run_id` — most recent run identifiers

---

## State Update Procedure (after each run)

1. Append new notice_ids to `seen_notice_ids`
2. Upsert `psc_benchmark_cache` with newly fetched PSC data
3. Append bid amounts to `historical_bid_amounts`
4. Upsert `competitor_registry` with new USASpending recipients
5. Increment all lifetime counters
6. Update `last_run_date` and `last_run_id`
7. Emit `state_updated` log entry

---

## Data Retention Policy

| Data Type | Retention |
|-----------|-----------|
| Audit log entries | 7 years (FAR 52.204-21) |
| Bid package text | 3 years |
| Opportunity records | 3 years |
| PSC benchmark cache | 7-day rolling TTL |
| `seen_notice_ids` | Indefinite |
| `competitor_registry` | Indefinite |

---

## Configuration

| Config Key | Value | Required | Notes |
|------------|-------|----------|-------|
| `AUDIT_RETENTION_YEARS` | `7` | Yes | FAR 52.204-21 requirement |
| `BID_RETENTION_YEARS` | `3` | Yes | Bid package retention |
| `PSC_CACHE_TTL_DAYS` | `7` | No | Benchmark cache TTL |
| `MAX_SEEN_IDS_IN_MEMORY` | `10000` | No | Cap before archiving |

---

## Standalone Skill Prompt (copy into Fleet)

SKILL: AUDIT TRAIL AND STATE MANAGEMENT

You are the Audit Trail and State Management module for the Nestova Federal
Bid Agent. You operate continuously throughout all pipeline runs.

1. For every action taken by any skill, record a log entry with: log_id (UUID),
   run_id (shared UUID for the run), timestamp, skill_name, event_type,
   entity_id, action_detail, result (success/skipped/failed/flagged), and
   ai_reasoning_summary (one sentence).

2. Maintain persistent memory: seen_notice_ids (deduplication), psc_benchmark_cache
   (7-day TTL), historical_bid_amounts, competitor_registry, and lifetime counters.

3. After each run: update all state keys, compile Daily Audit Trail Report,
   retain entries 7 years per FAR 52.204-21. Emit state_updated log entry
   after every memory write.

---

*Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
