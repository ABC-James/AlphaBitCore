# Node: analyze_risk_disclosures

| Property | Value |
|----------|-------|
| **ID** | `analyze_risk_disclosures` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3000` |
| **Position** | x: 800, y: 800 |

## Purpose

Ensures that marketing content contains risk disclosures proportionate to the complexity and risk profile of the assets being promoted. Uses the asset complexity classification from `classify_asset_class` to calibrate the minimum required disclosure level.

## Execution Guard

Executes **only when**:
- `{{classify_request_type.is_compliance_request}} == true` **AND**
- `{{classify_content_complexity.requires_risk_analysis}} == true` OR `{{classify_asset_class.highest_complexity}}` is `HIGH` or `VERY_HIGH`

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.firm_type}}` | Firm registration type |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |
| `{{classify_request_type.content_extracted}}` | The marketing content |
| `{{classify_asset_class.highest_complexity}}` | Asset complexity level |

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
  "requires_loss_of_principal_warning": true | false,
  "requires_illiquidity_warning": true | false,
  "requires_suitability_disclosure": true | false
}
```

## Risk Disclosure Requirements by Asset Complexity

### LOW / MODERATE Assets
**Minimum:** *"All investments involve risk, including possible loss of principal. Past performance does not guarantee future results."*

### HIGH Assets (Options, Derivatives, Leveraged, Alternatives)
Required disclosures:
- Substantial risk of loss (must be prominent)
- "Not suitable for all investors"
- Suitability considerations
- Liquidity risks (if applicable)
- Leverage disclosure and loss amplification
- Complexity warnings

### VERY HIGH Assets (Hedge Funds, Private Placements)
Required disclosures (must be prominent):
- Risk of total loss
- Illiquidity and lock-up periods
- Lack of daily pricing
- Accredited investor requirements (if retail audience)
- Lack of SEC registration (if applicable)
- Complex fee structure and its impact on returns
- Leverage and derivative risks
- Limited transparency relative to registered funds

## Fair and Balanced Principle (FINRA Rule 2210)

Risk disclosures must be:
- Proportional to the benefits claimed
- Not buried in fine print
- Clearly and prominently placed
- Given appropriate emphasis

## Severity

| Severity | Examples |
|----------|---------|
| **HIGH** | Performance presented with no risk language; misleading risk downplaying; HIGH/VERY HIGH assets with no disclosure |
| **MEDIUM** | Vague "subject to risk" without specifics; missing required risk categories for the asset type |
| **LOW** | Existing disclosures could be strengthened |

## Applicable Rules

- FINRA Rule 2210(d)(1)(A) — Fair and Balanced
- FINRA Rule 2210(d)(1)(E) — Risk Disclosure
- SEC Rule 156

## Safe-Fail

- HIGH/VERY HIGH assets without strong risk disclosure → HIGH issue
- Performance discussed without any risk context → HIGH issue
- Vague/generic risk language → MEDIUM with specific improvement suggestions
