# Node: classify_asset_class

| Property | Value |
|----------|-------|
| **ID** | `classify_asset_class` |
| **Type** | `MODEL` |
| **Version** | `1.0.0` |
| **Position** | x: 450, y: 875 |

## Purpose

Identifies the asset classes referenced in the marketing content and determines the associated disclosure requirements and regulatory complexity. Also controls whether the hedge fund and private placement specialized analyzers are triggered.

## Execution Guard

Executes **only when** `{{classify_request_type.is_compliance_request}} == true`.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.content_extracted}}` | Extracted marketing text |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |

## Output Schema

```json
{
  "asset_classes_mentioned": ["list"],
  "highest_complexity": "MINIMAL | LOW | MODERATE | HIGH | VERY_HIGH",
  "requires_illiquidity_disclosure": true | false,
  "requires_accredited_investor_language": true | false,
  "requires_fee_disclosure": true | false,
  "requires_risk_of_loss_disclosure": true | false,
  "requires_lack_of_liquidity_warning": true | false,
  "requires_suitability_language": true | false,
  "run_hedge_fund_analyzer": true | false,
  "run_private_placement_analyzer": true | false,
  "reasoning": "brief explanation"
}
```

## Complexity Ladder

| Level | Asset Types |
|-------|-------------|
| `MINIMAL` | Cash, Money Market |
| `LOW` | ETFs, Index Funds |
| `MODERATE` | Mutual Funds, Stocks, Bonds |
| `HIGH` | Options, Derivatives, REITs, Alternatives |
| `VERY_HIGH` | Hedge Funds, Private Equity, Private Placements |

## Detection Patterns

| Pattern | Keywords |
|---------|----------|
| Hedge Fund | "hedge fund", "long/short", "absolute return", "2 and 20" |
| Private Placement | "Reg D", "506(b)", "506(c)", "unregistered offering", "PPM" |
| Options/Derivatives | "options", "futures", "swaps", "derivatives" |
| Leveraged | "2x", "3x", "leveraged ETF", "inverse ETF" |

## Audience-Based Disclosure Rules

| Condition | Requirement |
|-----------|-------------|
| Retail + `VERY_HIGH` complexity | `requires_accredited_investor_language = true` |
| Retail + `HIGH` complexity | `requires_suitability_language = true` |

## Analyzer Trigger Logic

| Flag | Triggers When |
|------|---------------|
| `run_hedge_fund_analyzer` | Explicit "hedge fund" OR alternative fund with performance fees |
| `run_private_placement_analyzer` | "Reg D" OR "private placement" OR "unregistered offering" |

## Edge Cases

| Scenario | Result |
|----------|--------|
| "Alternative investments" (vague) | `HIGH`, not `VERY_HIGH` |
| Multiple asset classes mentioned | Use the highest complexity level |
| "Private real estate fund" | `VERY_HIGH` |
| "For accredited investors only" | `VERY_HIGH` |

## Safe-Fail Rules

- Ambiguous asset class → Choose higher complexity
- Uncertain whether to trigger a specialized analyzer → Run it

## Downstream Edges

`highest_complexity` and `run_*` flags gate the `analyze_risk_disclosures`, `analyze_hedge_fund_disclosures`, and `analyze_private_placement_disclosures` nodes.
