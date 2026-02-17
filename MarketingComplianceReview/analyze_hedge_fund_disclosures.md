# Node: analyze_hedge_fund_disclosures

| Property | Value |
|----------|-------|
| **ID** | `analyze_hedge_fund_disclosures` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3000` |
| **Position** | x: 800, y: 1100 |

## Purpose

Validates that marketing content for hedge funds and similar alternative investment vehicles includes all required investor protection disclosures. This is a specialized analyzer that only runs when the asset classification step identifies hedge fund characteristics.

## Execution Guard

Executes **only when**:
- `{{classify_request_type.is_compliance_request}} == true` **AND**
- `{{classify_asset_class.run_hedge_fund_analyzer}} == true`

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.content_extracted}}` | The marketing content |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |

## Output Schema

```json
{
  "issues": [
    {
      "severity": "HIGH | MEDIUM | LOW",
      "issue": "specific issue",
      "explanation": "why required",
      "required_disclosure": "exact text needed"
    }
  ],
  "has_issues": true | false,
  "highest_severity": "HIGH | MEDIUM | LOW | NONE"
}
```

## Required Disclosures

### 1. Accredited Investor Requirement (HIGH — if retail audience)
Must state:
> *"This investment is available only to accredited investors as defined under Regulation D."*

### 2. Risk of Total Loss (HIGH)
Must include prominent language:
> *"Investors may lose their entire investment"* or *"Risk of total loss of capital"*

### 3. Illiquidity and Lock-Up Terms (HIGH)
Must disclose: lock-up periods, redemption restrictions, liquidity terms, and advance notice requirements.

Example: *"Initial 1-year lock-up. Quarterly redemptions thereafter with 60-day notice."*

### 4. Performance and Management Fees (MEDIUM)
Must disclose: management fee (e.g., 1.5%), performance/incentive fee (e.g., 20%), hurdle rate (if applicable), high water mark provision.

Example: *"Management fee: 1.5% annually. Performance allocation: 20% of profits over 8% hurdle with high water mark."*

### 5. Leverage Disclosure (MEDIUM)
If leverage is used:
> *"Fund may employ leverage up to 3:1, which will amplify both gains and losses."*

### 6. Lack of Public Market (HIGH)
Must warn:
> *"No public market exists for these securities. Transfer restrictions apply."*

### 7. Limited Transparency (MEDIUM)
Should note: less frequent reporting than mutual funds, limited regulatory oversight compared to registered funds.

## Complete Required Disclosure Example

> *"ABC Hedge Fund LP is available only to accredited investors and qualified purchasers. This is a speculative investment involving a high degree of risk, including risk of total loss of capital. Fund employs leverage up to 3:1 which will amplify losses. Initial 1-year lock-up with quarterly redemptions thereafter on 60-day notice. No public market for securities and transfer is restricted. Management fee: 1.5%. Performance allocation: 20% over 6% hurdle. Fund is not registered under the Investment Company Act of 1940."*

## Applicable Rules

- Securities Act Rule 506
- Investment Advisers Act of 1940

## Safe-Fail

- Retail audience + hedge fund → Require accredited investor language (HIGH issue if missing)
- Any hedge fund content → Verify total loss risk is disclosed
- Vague or absent fee disclosure → Flag for specific disclosure
