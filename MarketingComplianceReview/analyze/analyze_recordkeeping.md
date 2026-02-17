# Node: analyze_recordkeeping

| Property | Value |
|----------|-------|
| **ID** | `analyze_recordkeeping` |
| **Type** | `TOOL` |
| **Tool** | `OpenAI_chat` |
| **Model** | `gpt-4o` |
| **Temperature** | `0.3` |
| **Max Tokens** | `2000` |
| **Position** | x: 800, y: 350 |

## Purpose

Determines the regulatory classification of the marketing communication (Retail, Institutional, or Correspondence), the required retention period, whether principal pre-approval is needed, and whether a pre-use FINRA filing is required. Outputs metadata for compliance workflow tracking.

## Execution Guard

Executes **only when** `{{classify_request_type.is_compliance_request}} == true`.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.firm_type}}` | Broker-Dealer, RIA, or Dual Registrant |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |
| `{{classify_request_type.channel}}` | Communication channel |
| `{{classify_content_complexity.word_count}}` | Word count of the content |

## Output Schema

```json
{
  "communication_type": "Retail Communication | Institutional Communication | Correspondence",
  "requires_principal_approval": true | false,
  "requires_pre_use_filing": true | false,
  "retention_period_years": 3,
  "special_requirements": ["list"],
  "filing_category": "category or null"
}
```

## Communication Type Classification

### Retail Communication (FINRA Rule 2210(a)(6))
Distributed to **more than 25 retail investors** within any 30-day period.
- Channels: Social media, public websites, mass emails, advertisements, factsheets
- Requires: Principal approval BEFORE use, 3-year retention, potential FINRA pre-use filing

### Institutional Communication (FINRA Rule 2210(a)(3))
Distributed **only to institutional investors**.
- Requires: Supervision and review, 3-year retention
- Does **not** require: Pre-approval or pre-use filing

### Correspondence (FINRA Rule 2210(a)(2))
Written to **25 or fewer retail investors** within any 30-day period.
- Requires: Supervision, 3-year retention
- Does **not** require: Pre-approval or pre-use filing

## Retention Period

All communication types: **3 years** (SEC Rule 17a-4; IA Act Rule 204-2)

## Principal Approval Rules

**Always required:**
- All retail communications (>25 retail investors)
- All advertisements
- Public websites
- Social media posts

**Supervision only (pre-approval not required):**
- Institutional communications
- Correspondence

## Pre-Use Filing with FINRA (Rule 2210(c))

**Must file within 10 days if:**
1. Firm is in its first year (new member) — applies to all retail communications
2. Content involves specific products: investment companies, direct participation programs (DPPs), collateralized mortgage obligations (CMOs), or options

**Exceptions (no filing required):**
- Correspondence
- Institutional communications
- Previously filed material being reused

## Decision Logic

```
IF audience = Retail AND (channel = Social OR Email OR Website OR Factsheet OR Pitchbook):
  → communication_type = "Retail Communication"
  → requires_principal_approval = true
  → retention_period_years = 3

IF audience = Institutional:
  → communication_type = "Institutional Communication"
  → requires_principal_approval = false
  → retention_period_years = 3
```

## Edge Cases

| Scenario | Result |
|----------|--------|
| Email to 26 retail investors | Retail Communication (>25 threshold) |
| Email to 24 retail investors | Correspondence (<25 threshold) |
| Public-facing website | Retail Communication |
| Social media post | Retail Communication |

## Applicable Rules

- FINRA Rule 2210 — Communications with the Public
- SEC Rule 17a-4 — Broker-Dealer Recordkeeping
- IA Act Rule 204-2 — RIA Recordkeeping

## Safe-Fail

- Unclear whether retail or institutional → Default to Retail (more restrictive treatment)
- Uncertain about pre-use filing obligation → Add to `special_requirements` for manual review
