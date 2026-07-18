# Skill 06  Compliance Self-Check

**Category:** Compliance and Quality Assurance  
**Trigger:** Called by Skill 05  
**Version:** 1.0 | Nestova LLC | NAICS: 423430

---

## Purpose

Perform an autonomous AI-powered compliance review of each generated bid package against federal procurement standards. Automatically identify and correct issues before the package is sent for human review. Ensures zero FAR violations and complete vendor information in every submission.

---

## Input Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `bid_packages` | array | Yes | - | Complete bid packages from Skill 05 |
| `max_correction_cycles` | integer | No | `2` | Max auto-correction attempts before flagging |

---

## Execution Logic

### Step 6.1 - Compliance Review Prompt

SYSTEM PROMPT:
You are a federal procurement compliance officer reviewing bid packages
submitted by small businesses. Review the following bid package against
federal procurement standards and SAM.gov submission requirements.

COMPLIANCE CHECKLIST (return PASS or FAIL with findings for each):

VENDOR INFORMATION CHECKS:
  C1: UEI is present and correctly formatted (12 alphanumeric characters)
    C2: CAGE code is present (5 alphanumeric characters)
      C3: Company name matches SAM.gov registration exactly: Nestova LLC
        C4: NAICS code 423430 is stated and appropriate for the requirement
          C5: Physical address is complete (street, city, state)
            C6: Authorized signatory name and title are present

            PRICING CHECKS:
              P1: All CLIN unit prices are positive numbers greater than $0.00
                P2: Extended prices equal quantity x unit price for each line item
                  P3: Total bid amount equals sum of all extended prices
                    P4: No price appears unreasonably low (below estimated cost of goods)
                      P5: No price appears unreasonably high (above 3x market rate for product)

                      CERTIFICATION CHECKS:
                        R1: FAR 52.204-21 compliance is affirmed
                          R2: SAM.gov active registration is affirmed
                            R3: No debarment/suspension certification is present
                              R4: Small business status is affirmed
                                R5: Buy American Act acknowledgment is present

                                COMPLETENESS CHECKS:
                                  Q1: No section contains placeholder text ({braces}, [BRACKETS], TBD, N/A)
                                    Q2: Technical narrative is at least 150 words and specific to solicitation
                                      Q3: Delivery terms are stated in calendar days ARO
                                        Q4: All six document sections are present
                                              (Cover, Certifications, Technical, Price, Delivery, Signature)

                                              Return ONLY valid JSON:
                                              {
                                                "overall_result": "PASS|FAIL",
                                                  "checks_passed": 20,
                                                    "checks_failed": 0,
                                                      "failed_checks": [
                                                          {
                                                                "check_id": "P2",
                                                                      "finding": "Extended price for CLIN 0003 is incorrect",
                                                                            "correction": "Multiply 5 x $149.99 = $749.95"
                                                                                }
                                                                                  ],
                                                                                    "corrections_applied": [],
                                                                                      "compliance_score": 100,
                                                                                        "ready_for_submission": true,
                                                                                          "reviewer_notes": "Package meets all federal procurement standards."
                                                                                          }

                                                                                          Bid Package to Review: {full_bid_package_text}

                                                                                          ### Step 6.2 - Auto-Correction Loop

                                                                                          For each failed check:
                                                                                            If correction is mathematical (wrong extended price, wrong total):
                                                                                                Apply the exact arithmetic correction automatically
                                                                                                  If correction is textual (placeholder text remaining):
                                                                                                      Re-generate that section only (do NOT regenerate entire document)
                                                                                                        If correction requires human judgment (scope ambiguity, missing data):
                                                                                                            Flag for human review - do NOT block pipeline
                                                                                                            
                                                                                                            After corrections:
                                                                                                              Increment correction_cycle_count
                                                                                                                If cycle_count > max_correction_cycles: re-run Step 6.1
                                                                                                                  If cycle_count >= max_correction_cycles AND still FAIL:
                                                                                                                      Set status: needs_human_review
                                                                                                                          Continue pipeline - do NOT halt
                                                                                                                          
                                                                                                                          ### Step 6.3 - Final Status Assignment
                                                                                                                          
                                                                                                                          overall_result = PASS  ->  bid_package.status = compliance_passed
                                                                                                                          overall_result = FAIL after max cycles  ->  bid_package.status = needs_human_review
                                                                                                                          
                                                                                                                          ### Step 6.4 - Compliance Log Entry
                                                                                                                          
                                                                                                                          Write per package:
                                                                                                                          {
                                                                                                                            "bid_id": "{bid_id}",
                                                                                                                              "notice_id": "{notice_id}",
                                                                                                                                "overall_result": "PASS|FAIL",
                                                                                                                                  "compliance_score": 0,
                                                                                                                                    "checks_passed": 0,
                                                                                                                                      "checks_failed": 0,
                                                                                                                                        "failed_checks": [],
                                                                                                                                          "corrections_applied": [],
                                                                                                                                            "correction_cycles": 0,
                                                                                                                                              "timestamp": "{ISO 8601}",
                                                                                                                                                "reviewer": "AI Compliance Module v1.0"
                                                                                                                                                }
                                                                                                                                                
                                                                                                                                                ---
                                                                                                                                                
                                                                                                                                                ## Output Schema
                                                                                                                                                
                                                                                                                                                | Field | Type | Description |
                                                                                                                                                |-------|------|-------------|
                                                                                                                                                | `compliant_packages` | array | Packages with status: compliance_passed |
                                                                                                                                                | `flagged_packages` | array | Packages needing human review |
                                                                                                                                                | `compliance_report` | object | total_reviewed, total_passed, total_flagged, common_failure_patterns |
                                                                                                                                                
                                                                                                                                                ---
                                                                                                                                                
                                                                                                                                                ## Error Handling
                                                                                                                                                
                                                                                                                                                | Error | Agent Behavior | Retry Strategy |
                                                                                                                                                |-------|---------------|----------------|
                                                                                                                                                | Compliance check times out | AI processing error | Re-run once; if fails again, set needs_human_review |
                                                                                                                                                | All packages fail compliance | Systemic issue | Alert william@nestova.net immediately with full failure detail |
                                                                                                                                                | Math correction creates new error | Cascade error | Log, revert to pre-correction version, flag for human review |
                                                                                                                                                
                                                                                                                                                ---
                                                                                                                                                
                                                                                                                                                ## Configuration
                                                                                                                                                
                                                                                                                                                | Config Key | Value | Required | Notes |
                                                                                                                                                |------------|-------|----------|-------|
                                                                                                                                                | `MAX_CORRECTION_CYCLES` | `2` | No | Max auto-correction iterations |
                                                                                                                                                | `AUTO_CORRECT_MATH` | `true` | No | Auto-fix arithmetic calculation errors |
                                                                                                                                                | `BLOCK_ON_FAIL` | `false` | No | If true, halts pipeline on any failure (default: flag and continue) |
                                                                                                                                                
                                                                                                                                                ---
                                                                                                                                                
                                                                                                                                                ## Standalone Skill Prompt (copy into Fleet)
                                                                                                                                                
                                                                                                                                                SKILL: COMPLIANCE SELF-CHECK
                                                                                                                                                
                                                                                                                                                You are the Compliance Self-Check module for the Nestova Federal Bid Agent.
                                                                                                                                                For each bid package from Skill 05:
                                                                                                                                                
                                                                                                                                                1. Run the 20-point compliance checklist across four categories:
                                                                                                                                                   - Vendor Information (C1-C6): UEI, CAGE, company name, NAICS, address,
                                                                                                                                                        signatory
                                                                                                                                                           - Pricing (P1-P5): positive prices, correct math, reasonable price points
                                                                                                                                                              - Certifications (R1-R5): FAR 52.204-21, SAM active, no debarment,
                                                                                                                                                                   small business, Buy American
                                                                                                                                                                      - Completeness (Q1-Q4): no placeholders, narrative at least 150 words,
                                                                                                                                                                           delivery terms stated, all 6 sections present
                                                                                                                                                                           
                                                                                                                                                                           2. For each failed check:
                                                                                                                                                                              - Mathematical errors: auto-correct
                                                                                                                                                                                 - Textual placeholders: re-generate that section only
                                                                                                                                                                                    - Human judgment required: flag, do not block
                                                                                                                                                                                    
                                                                                                                                                                                    3. Re-run compliance check after corrections (max 2 cycles).
                                                                                                                                                                                    
                                                                                                                                                                                    4. Set package status:
                                                                                                                                                                                       - PASS: compliance_passed - advance to Skill 07
                                                                                                                                                                                          - FAIL after max cycles: needs_human_review - flag in notification
                                                                                                                                                                                          
                                                                                                                                                                                          5. Write compliance log entry per package with full check results.
                                                                                                                                                                                             Return compliant_packages, flagged_packages, and compliance_report.
                                                                                                                                                                                             
                                                                                                                                                                                             ---
                                                                                                                                                                                             
                                                                                                                                                                                             *Nestova LLC | UEI: N47XNMUA9369 | CAGE: 21GW7 | v1.0 | July 2026*
