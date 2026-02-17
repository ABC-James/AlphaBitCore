# Marketing Compliance Workflow

An AI-powered workflow for reviewing financial services marketing and advertising content against FINRA Rule 2210 and SEC regulations.

## Overview

This workflow classifies incoming requests, routes them through specialized compliance analyzers, synthesizes all findings, and returns a structured JSON compliance report.

## Node Index

### Stage 1 — Request Classification
| Node | Type | Description |
|------|------|-------------|
| [classify_request_type](./classify/classify_request_type.md) | MODEL | Determines if the request contains reviewable marketing content and extracts metadata |
| [fast_fail_handler](./system/fast_fail_handler.md) | MODEL | Generates a polite rejection for non-compliance requests |

### Stage 2 — Content Classification (run in parallel)
| Node | Type | Description |
|------|------|-------------|
| [classify_content_complexity](./classify/classify_content_complexity.md) | MODEL | Measures content length, claim types, and which analyzers are required |
| [classify_asset_class](./classify/classify_asset_class.md) | MODEL | Identifies asset classes and triggers specialized disclosure analyzers |

### Stage 3 — Compliance Analyzers (run in parallel, conditionally)
| Node | Type | Model | Description |
|------|------|-------|-------------|
| [analyze_prohibited_statements](./analyze/analyze_prohibited_statements.md) | TOOL | Claude | Detects guarantees, risk-free claims, and other prohibited language |
| [analyze_performance_claims](./analyze/analyze_performance_claims.md) | TOOL | Claude | Validates performance presentation requirements (periods, net/gross, disclaimers) |
| [analyze_risk_disclosures](./analyze/analyze_risk_disclosures.md) | TOOL | Claude | Checks adequacy of risk disclosures relative to asset complexity |
| [analyze_testimonials](./analyze/analyze_testimonials.md) | TOOL | Claude | Reviews testimonial and endorsement disclosure compliance |
| [analyze_communications_standards](./analyze/analyze_communications_standards.md) | TOOL | Claude | Assesses fair and balanced presentation, material omissions |
| [analyze_recordkeeping](./analyze/analyze_recordkeeping.md) | TOOL | OpenAI | Determines communication type, principal approval, and retention requirements |
| [analyze_hedge_fund_disclosures](./analyze/analyze_hedge_fund_disclosures.md) | TOOL | Claude | Validates hedge fund-specific disclosure requirements |
| [analyze_private_placement_disclosures](./analyze/analyze_private_placement_disclosures.md) | TOOL | Claude | Validates Regulation D private placement disclosure requirements |

### Stage 4 — Synthesis & Output
| Node | Type | Model | Description |
|------|------|-------|-------------|
| [synthesize_results](./system/synthesize_results.md) | TOOL | OpenAI | Aggregates all analyzer findings into a unified compliance verdict |
| [format_output](./system/format_output.md) | MODEL | — | Formats the final structured JSON response |

## Data Flow

```
classify_request_type
  ├── [false] → fast_fail_handler → format_output
  └── [true]  → classify_content_complexity ──┐
               → classify_asset_class         ├──> analyze_* (parallel)
                                              │         ↓
                                              └──> synthesize_results → format_output
```

## Output Schema

The workflow returns one of two JSON structures:

- **`REJECTED_REQUEST`** — when the input is not a compliance review request
- **`COMPLIANCE_REVIEW`** — a full compliance report including overall risk (`LOW/MEDIUM/HIGH/CRITICAL`), status (`ACCEPT/NEEDS CHANGES/REJECT/ESCALATION REQUIRED`), categorized issues, required edits, required disclosures, substantiation needed, and recordkeeping metadata

## Regulatory Coverage

- FINRA Rule 2210 — Communications with the Public
- SEC Rule 10b-5 — Prohibition on Fraud
- SEC Rule 156 — Investment Company Sales Literature
- SEC Rule 206(4)-1 — Investment Adviser Marketing
- Securities Act Rule 506 / Regulation D
- SEC Rule 17a-4 / IA Act Rule 204-2 — Recordkeeping
