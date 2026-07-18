# Skill 05  Bid Package Generation

**Category:** Document Generation  
**Trigger:** Called by Skill 04  
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Generate a complete, professional, government-ready bid package document for each qualified opportunity. Produces an SF-1449-formatted document containing all required sections: cover page, representations and certifications, technical narrative, price schedule, delivery terms, and signature block. Every field must be fully populated  zero placeholder text.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `sourced_opportunities` | array | Yes | - | Opportunities with line items from Skill 04 |
| `document_format` | string | No | `structured_text` | Output format: markdown or structured_text |

---

## Execution Logic

### Step 5.1 - Technical Narrative Generation

SYSTEM PROMPT:
You are a professional federal proposal writer retained by Nestova LLC.
Write a compelling Technical Approach and Capability Statement.

Writing Requirements:
- Government-grade formal tone, precise and confident
- - Three complete paragraphs:
  -   (1) Nestova capability aligned to NAICS 423430 requirement
  -     (2) Product sourcing methodology and supply chain reliability
  -   (3) Delivery commitment and low-risk value proposition
  -   - No placeholder text, no brackets, no template language
      - - Maximum 350 words total
       
        - Company Profile:
        -   Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | NAICS: 423430
        -     1152 Rock Springs Rd, Blacksburg, SC
        -   supply.nestova.net | william@nestova.net
        -     Authorized Signatory: William Jackson, Managing Member
       
        - Solicitation Context:
        -   Title: {title}
        -     Agency: {agency}
        -   Products Required: {product_names_from_line_items}
        -     Total Bid Amount: ${total_bid_amount}
        -   Delivery Commitment: {delivery_days_aro} calendar days ARO
       
        -   ### Step 5.2 - Full Document Assembly
       
        -   Assemble the complete bid package with this exact structure:
       
        -   SECTION A - COVER PAGE
        -     Solicitation Number, Title, Issuing Agency, Date of Issuance, Deadline
        -   NAICS Code: 423430
        -     Offeror: Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7
        -   Address: 1152 Rock Springs Rd, Blacksburg, SC
        -     POC: William Jackson, Managing Member | william@nestova.net
        -   TOTAL OFFERED PRICE: ${total_bid_amount}
       
        -   SECTION B - REPRESENTATIONS AND CERTIFICATIONS
        -     [CERTIFIED] Active SAM.gov registration - UEI: N47XNMUA9369
        -   [CERTIFIED] Small Business - FAR 19.001, NAICS 423430
        -     [CERTIFIED] Not debarred or suspended - FAR 9.4
        -   [CERTIFIED] FAR 52.204-21 - COMPLIANT
        -     [CERTIFIED] FAR 52.212-3 - COMPLIANT
        -   [CERTIFIED] Buy American Act - COMPLIANT
        -     [CERTIFIED] No organizational conflicts of interest
       
        - SECTION C - TECHNICAL APPROACH AND CAPABILITY STATEMENT
        -   {AI-generated narrative from Step 5.1}
       
        -   SECTION D - PRICE AND COST SCHEDULE (SF-1449 FORMAT)
        -     Full CLIN line-item table from Skill 04
        -   TOTAL OFFERED PRICE: ${total_bid_amount}
       
        -   SECTION E - DELIVERY AND CONTRACT TERMS
        -     Delivery: {delivery_days_aro} calendar days ARO, FOB Destination
        -   Payment: Net 30 days from receipt of proper invoice
        -     Warranty: Manufacturer standard commercial warranty
        -   Inspection/Acceptance: Destination
       
        -   SECTION F - AUTHORIZED SIGNATURE BLOCK
        -     Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7
        -   William Jackson, Managing Member
        -     william@nestova.net | {today}
       
        - ### Step 5.3 - Placeholder Sweep
       
        - Scan assembled document for remaining {curly braces}, [BRACKET] markers,
        - or literal TBD/N/A in substantive fields. Re-populate using available data.
        - Document must contain zero placeholder text before advancing to Skill 06.
       
        - ---

        ## Output Schema

        | Field | Type | Description |
        |-------|------|-------------|
        | `bid_packages` | array | Complete bid package objects with full document text |
        | `generation_summary` | object | packages_generated, total_pipeline_value, avg_bid_amount, avg_word_count |

        ---

        ## Error Handling

        | Error | Agent Behavior | Retry Strategy |
        |-------|---------------|----------------|
        | AI narrative contains placeholder text | Sweep detects issue | Re-generate narrative only, retry once |
        | Line item table truncated | Incomplete output | Re-generate price schedule with explicit count |
        | Missing solicitation number | Not in SAM.gov | Use notice_id as reference, note in header |
        | Technical narrative under 100 words | Quality threshold | Re-prompt with minimum 200-word requirement |

        ---

        ## Configuration

        | Config Key | Value | Required | Notes |
        |------------|-------|----------|-------|
        | `COMPANY_NAME` | `Nestova LLC` | Yes | Legal company name |
        | `UEI` | `N47XNMUA9369` | Yes | SAM.gov unique entity identifier |
        | `CAGE` | `21GW7` | Yes | CAGE code |
        | `ADDRESS` | `1152 Rock Springs Rd, Blacksburg, SC` | Yes | Physical address |
        | `CONTACT_EMAIL` | `william@nestova.net` | Yes | Primary contact email |
        | `WEBSITE` | `supply.nestova.net` | Yes | Company website |
        | `AUTHORIZED_SIGNATORY` | `William Jackson` | Yes | Authorized offeror name |
        | `SIGNATORY_TITLE` | `Managing Member` | Yes | Title of authorized offeror |

        ---

        ## Standalone Skill Prompt (copy into Fleet)

        SKILL: BID PACKAGE GENERATION

        You are the Bid Package Generation module for the Nestova Federal Bid Agent.
        For each sourced opportunity:

        1. Generate a Technical Approach narrative (3 paragraphs, max 350 words):
        2.    (a) Nestova capability aligned to NAICS 423430 requirement
        3.   (b) Product sourcing methodology and supply chain reliability
        4.      (c) Delivery commitment and low-risk value proposition
        5.     Professional government tone, no placeholders, fully specific.
       
        6. 2. Assemble the complete SF-1449-format bid package with 6 sections:
           3.    A) Cover Page with all Nestova identity fields and TOTAL OFFERED PRICE
           4.   B) Representations and Certifications (FAR 52.204-21, 52.212-3, SAM
           5.         active, small business, Buy American, no conflicts)
           6.        C) Technical Approach narrative from step 1
           7.       D) Price/Cost Schedule - full CLIN line-item table from Skill 04
           8.      E) Delivery and Contract Terms (FOB Destination, Net 30, warranty)
           9.     F) Authorized Signature Block - William Jackson, Managing Member
          
           10. 3. Run placeholder sweep - confirm zero {curly braces}, [brackets], TBD,
               4.    or N/A remain in substantive fields. Re-generate any incomplete section.
              
               5.4. Return bid_packages array with complete document text per opportunity.

               Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7
               1152 Rock Springs Rd, Blacksburg, SC
               william@nestova.net | supply.nestova.net
               Authorized: William Jackson, Managing Member

               ---

               *Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
