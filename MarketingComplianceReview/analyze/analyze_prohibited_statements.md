# Node: analyze_prohibited_statements

| Property | Value |
|----------|-------|
| **ID** | `analyze_prohibited_statements` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3000` |
| **Position** | x: 800, y: 500 |

## Purpose

Detects prohibited, restricted, and problematic language in financial marketing content under FINRA Rule 2210 and SEC regulations. Critically distinguishes between **standard marketing puffery** (permitted) and **substantive investment claims** (regulated).

## Execution Guard

Executes **only when** `{{classify_request_type.is_compliance_request}} == true`.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.firm_type}}` | Broker-Dealer, RIA, Dual Registrant, or Unknown |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |
| `{{classify_request_type.content_extracted}}` | The marketing content to analyze |

## Output Schema

```json
{
  "issues": [
    {
      "severity": "HIGH | MEDIUM | LOW",
      "issue": "specific prohibited statement",
      "rule_violated": "FINRA Rule 2210 | SEC Rule",
      "explanation": "why prohibited",
      "suggested_alternative": "compliant wording"
    }
  ],
  "has_issues": true | false,
  "highest_severity": "HIGH | MEDIUM | LOW | NONE"
}
```

## Permitted Language (Do Not Flag)

The following categories are **always acceptable** and should never be flagged:

**Service/Firm Quality Superlatives:** "world-class service", "exceptional team", "dedicated professionals", "innovative approach", "trusted partner", "bespoke portfolios", "rigorous process"

**Aspirational/Mission Language:** "helping clients achieve their goals", "building long-term wealth", "putting clients first"

**Conditional/Aspirational Performance (with appropriate disclaimers):** "seeking to outperform", "aiming for strong risk-adjusted returns", "targeting attractive returns"

## Severity Classification

### HIGH — Absolutely Prohibited
- Guarantees of investment returns ("guaranteed", "promise", "assure" + return/profit)
- Risk misrepresentation ("no risk", "risk-free", "safe investment")
- False regulatory claims ("SEC approved", "FINRA endorsed")
- Promises of specific future returns ("will return X%")

### MEDIUM — Restricted, Requires Revision
- Unsubstantiated performance rankings ("#1 fund", "best returns") without a data source
- Absolute statements about investment outcomes ("always outperforms", "never loses money")
- Promissory investment language ("will outperform the market")
- Specific future projections stated as fact ("will achieve 10%")

> **Note:** Superlatives about **service quality** are NOT MEDIUM — they are puffery and permitted.

### LOW — Minor, Informational Only
- Vague performance claims alongside actual numbers that need specifics
- Terminology that could be clarified for retail audiences
- Minor promotional enthusiasm well short of substantive claims

## Key Distinction: Puffery vs. Substantive Claims

| Type | Example | Treatment |
|------|---------|-----------|
| Puffery | "Exceptional risk management" | Permitted — subjective opinion |
| Substantive | "Best risk-adjusted returns in our peer group" | MEDIUM — verifiable, needs proof |
| Puffery | "We've delivered strong results for clients" | Permitted — general statement |
| Substantive | "We've returned 15% annually for 10 years" | HIGH (without disclaimer) |

## Applicable Rules

- FINRA Rule 2210 — Communications with the Public
- SEC Rule 10b-5 — Prohibition on Fraud
- SEC Rule 156 — Investment Company Sales Literature

## Safe-Fail

- Uncertain if prohibited → Is it a substantive, verifiable claim or puffery?
  - Puffery → Do not flag
  - Substantive claim without support → Flag as MEDIUM
  - Vague promotional language with no specific claim → Flag as LOW only if it could genuinely mislead
