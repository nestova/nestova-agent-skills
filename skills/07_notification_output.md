# Skill 07  Notification and Output

**Category:** Output and Delivery  
**Trigger:** Called by Skill 06  
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Package all compliant bid packages into a structured daily output report and deliver them to william@nestova.net via SendGrid. Generate a BID SUMMARY CARD for each package and a full run summary. Handle the No Opportunities Today notification path gracefully.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `compliant_packages` | array | Yes | - | Compliance-checked packages from Skill 06 |
| `flagged_packages` | array | No | [] | Packages needing human review |
| `run_statistics` | object | Yes | - | Aggregated stats from all upstream skills |
| `no_opportunities_today` | boolean | No | false | Flag from Skill 01 |

---

## Execution Logic

### Step 7.1 - BID SUMMARY CARD Generation

Generate one structured summary card per compliant package:

  Solicitation: {solicitation_number}
    Title: {title}
      Agency: {agency}
        Deadline: {deadline}
          TOTAL BID AMOUNT: ${total_bid_amount}
            Bid-Fit Score: {score} ({confidence})
              Pricing Strategy: {pricing_strategy}
                Benchmark Range: ${benchmark_low} - ${benchmark_high}
                  Blended Margin: {blended_margin_pct}%
                    TOP COMPETITOR: {top_competitor_name}
                      WIN PROBABILITY: {win_probability_at_mid}
                        COMPLIANCE: {compliance_result} (Score: {compliance_score}/100)
                          STATUS: READY FOR YOUR REVIEW AND SUBMISSION
                            Line Items: {line_item_count}
                              Delivery: {delivery_days_aro} calendar days ARO
                                Sourcing: {sourcing_confidence} confidence
                                  Solicitation URL: {ui_link}

                                  ### Step 7.2 - Run Statistics Block

                                    DAILY RUN SUMMARY - {run_date}
                                      Run Completed: {run_timestamp}
                                        Opportunities Found: {total_discovered}
                                          Passed Filters: {total_passed_filters}
                                            Scored by AI: {total_scored}
                                              Passed Scoring (>=0.75): {total_passed_scoring}
                                                Bid Packages Built: {packages_generated}
                                                  Compliance Passed: {compliance_passed}
                                                    Flagged for Review: {flagged_count}
                                                      Total Pipeline Value: ${total_pipeline_bid_value}

                                                      ### Step 7.3 - Email Composition

                                                      When opportunities exist (no_opportunities_today = false):

                                                        SendGrid API: POST https://api.sendgrid.com/v3/mail/send
                                                          Authorization: Bearer {SENDGRID_API_KEY}
                                                            From: noreply@nestova.net (display name: Nestova Bid Agent)
                                                              To: william@nestova.net
                                                                Subject: [Nestova Agent] {compliant_count} Bid Package(s) Ready - {run_date}

                                                                  Body structure:
                                                                      1. Run Statistics Block (Step 7.2)
                                                                          2. One BID SUMMARY CARD per compliant package (Step 7.1)
                                                                              3. Full bid package document text below each summary card
                                                                                  4. If flagged_packages exist: list each with title, agency, bid amount,
                                                                                         and compliance failure reason
                                                                                             5. Footer: You are the final decision-maker on all bids.
                                                                                                    This agent does not submit bids autonomously.
                                                                                                           Nestova Federal Bid Agent | supply.nestova.net
                                                                                                           
                                                                                                           When no opportunities (no_opportunities_today = true):
                                                                                                           
                                                                                                             Subject: [Nestova Agent] No New Opportunities Today - {run_date}
                                                                                                             
                                                                                                               Body: The Nestova Federal Bid Agent completed its daily SAM.gov scan and
                                                                                                                 found no new opportunities matching NAICS 423430. Filter summary:
                                                                                                                     NAICS Code: 423430
                                                                                                                         Value Range: $2,500 - $5,000,000
                                                                                                                             Deadline Buffer: 72+ hours required
                                                                                                                                 Deduplication: {total_already_seen} previously seen opportunities excluded
                                                                                                                                   The agent will run again tomorrow at 06:00 AM Eastern Time.
                                                                                                                                   
                                                                                                                                   ### Step 7.4 - Audit Trail Payload
                                                                                                                                   
                                                                                                                                   Pass final run audit record to Skill 08:
                                                                                                                                     run_date, run_timestamp, total_discovered, total_passed_filters,
                                                                                                                                       total_scored, total_passed_scoring, packages_generated, compliance_passed,
                                                                                                                                         flagged_count, total_pipeline_value, email_delivered, status: run_complete
                                                                                                                                         
                                                                                                                                         ---
                                                                                                                                         
                                                                                                                                         ## Output Schema
                                                                                                                                         
                                                                                                                                         | Field | Type | Description |
                                                                                                                                         |-------|------|-------------|
                                                                                                                                         | `email_status` | string | sent or failed |
                                                                                                                                         | `packages_delivered` | integer | Count of packages in email |
                                                                                                                                         | `notification_type` | string | opportunities_found or no_opportunities |
                                                                                                                                         | `sendgrid_message_id` | string | SendGrid message ID for tracking |
                                                                                                                                         | `run_audit_payload` | object | Final run record for Skill 08 |
                                                                                                                                         
                                                                                                                                         ---
                                                                                                                                         
                                                                                                                                         ## Error Handling
                                                                                                                                         
                                                                                                                                         | Error | Agent Behavior | Retry Strategy |
                                                                                                                                         |-------|---------------|----------------|
                                                                                                                                         | SendGrid 401 | API key invalid | Alert via secondary channel, log failure |
                                                                                                                                         | SendGrid 429 | Rate limit | Wait 10s, retry once |
                                                                                                                                         | SendGrid 500 | Server error | Retry after 30s, max 2 attempts, log if fails |
                                                                                                                                         | Zero compliant AND zero flagged | Unexpected empty | Send minimal run summary regardless |
                                                                                                                                         | Email body exceeds 20MB limit | Too large | Send summaries only, note truncation in email |
                                                                                                                                         
                                                                                                                                         ---
                                                                                                                                         
                                                                                                                                         ## Configuration
                                                                                                                                         
                                                                                                                                         | Config Key | Value/Secret | Required | Notes |
                                                                                                                                         |------------|-------------|----------|-------|
                                                                                                                                         | `SENDGRID_API_KEY` | Secret: SENDGRID_API_KEY | Yes | SendGrid v3 API key |
                                                                                                                                         | `NOTIFICATION_EMAIL` | `william@nestova.net` | Yes | Operator delivery address |
                                                                                                                                         | `FROM_EMAIL` | `noreply@nestova.net` | Yes | Verified SendGrid sender |
                                                                                                                                         | `FROM_NAME` | `Nestova Bid Agent` | No | Display name on emails |
                                                                                                                                         
                                                                                                                                         ---
                                                                                                                                         
                                                                                                                                         ## Standalone Skill Prompt (copy into Fleet)
                                                                                                                                         
                                                                                                                                         SKILL: NOTIFICATION AND OUTPUT
                                                                                                                                         
                                                                                                                                         You are the Notification and Output module for the Nestova Federal Bid Agent.
                                                                                                                                         
                                                                                                                                         1. For each compliant package, generate a BID SUMMARY CARD with: solicitation
                                                                                                                                            number, title, agency, deadline, total bid amount, bid-fit score, pricing
                                                                                                                                               strategy, benchmark range, blended margin, top competitor, win probability,
                                                                                                                                                  compliance score, line item count, and delivery timeline.
                                                                                                                                                  
                                                                                                                                                  2. Compile the Daily Run Statistics Block covering the full pipeline funnel:
                                                                                                                                                     discovered - filtered - scored - passed - packages - compliance passed.
                                                                                                                                                     
                                                                                                                                                     3. Compose and send the daily email via SendGrid (SENDGRID_API_KEY) to
                                                                                                                                                        william@nestova.net:
                                                                                                                                                           - Opportunities exist: subject [Nestova Agent] {N} Bid Package(s) Ready -
                                                                                                                                                                {date}, include run stats, all summary cards, full package text, and
                                                                                                                                                                     flagged packages section if any.
                                                                                                                                                                        - No opportunities: subject [Nestova Agent] No New Opportunities Today -
                                                                                                                                                                             {date}, include filter summary and next run time (06:00 AM Eastern).
                                                                                                                                                                             
                                                                                                                                                                             4. Always end email with: You are the final decision-maker on all bids.
                                                                                                                                                                                This agent does not submit bids autonomously.
                                                                                                                                                                                
                                                                                                                                                                                5. Pass final run_audit_payload to Skill 08. Return email_status and
                                                                                                                                                                                   packages_delivered count.
                                                                                                                                                                                   
                                                                                                                                                                                   ---
                                                                                                                                                                                   
                                                                                                                                                                                   *Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
