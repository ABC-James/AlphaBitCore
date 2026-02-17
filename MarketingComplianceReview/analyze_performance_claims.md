# Node: analyze_performance_claims

| Property | Value |
|----------|-------|
| **ID** | `analyze_performance_claims` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3500` |
| **Position** | x: 800, y: 650 |

## Purpose

Ensures that any investment performance data presented in the marketing content meets FINRA and SEC requirements — including standardized time periods, net-of-fees disclosure, past performance disclaimers, and proper benchmark identification.

## Execution Guard

Executes **only when**:
- `{{classify_request_type.is_compliance_request}} == true` **AND**
- `{{classify_content_complexity.requires_performance_analysis}} == true` OR `complexity` is `MODERATE` or `COMPREHENSIVE`

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.firm_type}}` | Firm registration type |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |
| `{{classify_request_type.content_extracted}}` | The marketing content |

## Output Schema

```json
{
  "issues": [
    {
      "severity": "HIGH | MEDIUM | LOW",
      "issue": "specific issue",
      "rule_violated": "rule",
      "explanation": "why required",
      "required_disclosure": "exact text needed"
    }
  ],
  "has_issues": true | false,
  "highest_severity": "HIGH | MEDIUM | LOW | NONE",
  "requires_standardized_performance": true | false,
  "requires_benchmark_disclosure": true | false,
  "requires_net_of_fees_disclosure": true | false,
  "requires_past_performance_disclaimer": true | false
}
```

## Required Elements

### 1. Standardized Time Periods (FINRA Rule 2210)
Must show: **1-year, 5-year, 10-year, and since inception**
- Exception: If fund is less than 1 year old, inception-only is acceptable
- Red flags: Only showing the best period; using custom date ranges without standard periods

### 2. Net of Fees (SEC/FINRA)
Must state: Is return gross or net? Which fees are deducted? What is the impact if gross?
- Red flag: No fee mention, or vague "net of fees" without specification

### 3. Past Performance Disclaimer
Required: *"Past performance does not guarantee future results"*
- Red flag: Performance shown without this disclaimer

### 4. Benchmark (if comparison is made)
Must include: Full benchmark name, Total Return vs Price Index distinction, reason it is an appropriate comparison
- Red flag: "vs benchmark" without naming it; using an inappropriate benchmark

### 5. Ranking/Quartile Claims
Must include: Data source, peer group definition, time period, number of funds in universe
- Red flag: "Top quartile" without a source

## Severity

| Severity | Examples |
|----------|---------|
| **HIGH** | Performance without any disclaimers; cherry-picked periods; gross performance misrepresented as net |
| **MEDIUM** | Missing standard time periods; incomplete benchmark identification; vague fee disclosure |
| **LOW** | Could strengthen disclaimers; minor formatting inconsistencies |

## Acceptable vs. Problematic Examples

✅ `"XYZ Fund returned 12.5% in 2024 (net), vs S&P 500 Total Return 11.2%. Past performance does not guarantee future results."`

❌ `"15% return!"` — missing period, net/gross designation, and disclaimer

❌ `"Beat the market"` — which market? over what period?

## Applicable Rules

- FINRA Rule 2210(d)(1)(F)
- SEC Rule 156
- SEC Rule 206(4)-1

## Safe-Fail

- Performance without numbers → Flag as needing specifics or removal
- Ambiguous time period → MEDIUM issue
- Weak disclaimers → LOW with suggested strengthening
