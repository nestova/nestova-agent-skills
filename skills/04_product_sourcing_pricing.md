# Skill 04  Product Sourcing and Pricing

**Category:** Catalog Intelligence  
**Trigger:** Called by Skill 03  
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Use AI reasoning to identify the specific products needed for each solicitation, generate a complete CLIN-level line-item pricing table with Nestova's margin applied, and validate the total bid price falls within the competitive benchmark range from Skill 03.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `benchmarked_opportunities` | array | Yes |  | Benchmarked opportunities from Skill 03 |
| `default_margin_pct` | float | No | `22` | Default markup % above cost for standard items |
| `commodity_margin_pct` | float | No | `15` | Markup % for high-volume commodity items |
| `minimum_margin_pct` | float | No | `10` | Minimum acceptable blended margin |

---

## Execution Logic

### Step 4.1  Product Identification and Pricing Prompt

SYSTEM PROMPT:
You are a procurement specialist and IT product catalog expert for Nestova LLC,
a federal IT product reseller and merchant wholesaler (NAICS 423430).

Nestova sources products through these authorized federal distribution channels:
  - CDW-G (GSA Schedule holder)
  -   - Ingram Micro Federal
      -   - TD SYNNEX
          -   - SHI Government Solutions
              -   - PC Mall Gov
                  -   - Zones
                      -   - Direct manufacturer programs: Dell Technologies, HP Inc, Lenovo, Cisco
                          -     Systems, Logitech, Brother Industries, Epson America, Zebra Technologies
                       
                          - Solicitation Details:
                          - Title: {title}
                          - Agency: {agency}
                          - Description: {description}
                          - AI-Inferred Product Category: {product_category}
                          - Recommended Product Types: {recommended_products}
                          - Estimated Solicitation Value: {estimated_value}
                          - Competitive Benchmark Range: ${benchmark_low} to ${benchmark_high}
                          - Recommended Target Price: ${recommended_target_price}
                          - Pricing Strategy: {pricing_strategy}
                          - Win Probability at Target: {win_probability_at_mid}
                       
                          - Generate a complete CLIN-level line-item pricing table for this solicitation.
                       
                          - Rules:
                          - - Use realistic commercial pricing for 2025-2026 federal IT procurement market
                            - - Apply 22% margin above cost for standard items
                              - - Apply 15% margin for commodity or bulk items
                                - - Ensure TOTAL BID PRICE lands within competitive benchmark range above
                                  - - Use complete, accurate product names and realistic manufacturer part numbers
                                    - - Include realistic quantities matching the apparent scope of the solicitation
                                      - - Delivery lead times: standard IT products 3-10 business days; specialty 10-30 days
                                       
                                        - Return ONLY valid JSON with fields: line_items array, total_bid_amount, total_cost,
                                        - blended_margin_pct, within_benchmark, delivery_days_aro, sourcing_confidence,
                                        - sourcing_strategy.
                                       
                                        - ### Step 4.2  Mathematical Validation
                                       
                                        - Verify per line item:
                                        -   unit_price     = unit_cost x (1 + margin_pct / 100)
                                        -     extended_price = quantity x unit_price
                                       
                                        - Verify totals:
                                        -   total_bid_amount = SUM(extended_prices)
                                       
                                        -   Auto-correct any arithmetic errors found.
                                       
                                        -   ### Step 4.3  Margin Validation
                                       
                                        -   If blended_margin_pct > minimum_margin_pct (10%):
                                        -     Re-prompt AI to increase unit prices to achieve minimum margin
                                        -   Maximum 2 correction cycles
                                       
                                        -   ### Step 4.4  Benchmark Validation
                                       
                                        -   If total_bid_amount NOT between benchmark_low and benchmark_high:
                                        -     Re-prompt AI to adjust quantities or unit prices
                                        -   Maximum 2 correction cycles
                                        -     If still outside after 2 cycles: use benchmark midpoint, flag needs_price_review
                                       
                                        - ### Step 4.5  Lead Time Risk Flag
                                       
                                        - For each line item:
                                        -   If lead_time_days > 30 AND solicitation indicates urgent delivery:
                                        -       Set lead_time_risk = true on that line item
                                        -       Add sourcing note in bid package
                                       
                                        -   ---

                                        ## Output Schema

                                        | Field | Type | Description |
                                        |-------|------|-------------|
                                        | `sourced_opportunities` | array | Opportunities with validated line-item pricing tables attached |
                                        | `sourcing_summary` | object | total_line_items_generated, avg_blended_margin, total_pipeline_bid_value, items_with_lead_time_risk |

                                        ---

                                        ## Error Handling

                                        | Error | Agent Behavior | Retry Strategy |
                                        |-------|---------------|----------------|
                                        | Line item with unit_price = $0 | Validation failure | Re-prompt with explicit min price instruction, retry once |
                                        | Total bid outside benchmark after 2 cycles | Cannot converge | Use benchmark midpoint, flag needs_price_review |
                                        | Product category too vague | Insufficient data | Set sourcing_confidence=LOW, add disclaimer note |
                                        | Blended margin below floor after 2 cycles | Cannot meet minimum | Flag for human review, include margin warning in notification |

                                        ---

                                        ## Configuration

                                        | Config Key | Value | Required | Notes |
                                        |------------|-------|----------|-------|
                                        | `DEFAULT_MARGIN_PCT` | `22` | No | Standard item markup percentage |
                                        | `COMMODITY_MARGIN_PCT` | `15` | No | Bulk/commodity item markup |
                                        | `MINIMUM_MARGIN_PCT` | `10` | Yes | Floor  flag if blended margin falls below |
                                        | `MAX_LINE_ITEMS` | `20` | No | Maximum line items per bid package |

                                        ---

                                        ## Standalone Skill Prompt (copy into Fleet)

                                        SKILL: PRODUCT SOURCING AND PRICING

                                        You are the Product Sourcing and Pricing module for the Nestova Federal Bid
                                        Agent (NAICS 423430). For each benchmarked opportunity:

                                        1. Run the Product Identification prompt to generate a complete CLIN-level
                                        2.    line-item table. Nestova sources from CDW-G, Ingram Micro, TD SYNNEX,
                                        3.   SHI, Zones, and direct manufacturer programs. Apply 22% margin standard,
                                        4.      15% for commodity items. Total bid must land within benchmark range.
                                       
                                        5.  2. Validate all math: unit_price = unit_cost x (1 + margin/100),
                                            3.    extended_price = qty x unit_price, total = SUM(extended_prices).
                                            4.   Auto-correct any errors.
                                          
                                            5.   3. Validate blended_margin_pct >= 10%. If below, re-prompt AI to adjust
                                                 4.    pricing. Max 2 correction cycles.
                                              
                                                 5.4. Validate total_bid_amount within benchmark_low to benchmark_high.
                                                    If outside after 2 cycles, use midpoint, flag needs_price_review.

                                                 5. Flag any line item with lead_time_days > 30 where solicitation
                                                 6.    indicates urgent delivery.
                                              
                                                 7.6. Return sourced_opportunities with complete, validated line-item tables.

                                                 ---

                                                 *Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
