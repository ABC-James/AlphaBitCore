# Node: analyze_testimonials

| Property | Value |
|----------|-------|
| **ID** | `analyze_testimonials` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3000` |
| **Position** | x: 800, y: 200 |

## Purpose

Reviews marketing content for client testimonials, endorsements, and third-party ratings to ensure all required disclosures are present under FINRA Rule 2210 and SEC Investment Adviser Marketing Rule 206(4)-1.

## Execution Guard

Executes **only when**:
- `{{classify_request_type.is_compliance_request}} == true` **AND**
- `{{classify_content_complexity.requires_testimonial_analysis}} == true` OR `{{classify_content_complexity.has_testimonials}} == true`

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.firm_type}}` | Broker-Dealer, RIA, or Dual Registrant |
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
      "required_disclosure": "exact text"
    }
  ],
  "has_issues": true | false,
  "highest_severity": "HIGH | MEDIUM | LOW | NONE",
  "requires_compensation_disclosure": true | false,
  "requires_atypical_results_disclosure": true | false
}
```

## Required Disclosures

### 1. Compensation Disclosure (Required)
Must disclose whether the testimonial provider was compensated.

✅ `"Client was not compensated for this testimonial."`
✅ `"Client received a $50 gift card."`
❌ No compensation disclosure at all

### 2. Atypical Results Disclaimer
Required when specific results are mentioned.

Required language: *"Results not typical"* or *"Individual results may vary."*

### 3. Material Conflicts of Interest
Must disclose: family member relationships, affiliated parties, or any material relationship between the firm and the testimonial provider.

### 4. No Regulatory Implication
Cannot imply SEC or FINRA endorsement through the use of testimonials.

### 5. Third-Party Ratings and Awards
Must disclose: who granted the award, the selection criteria, the date of the award, and whether the firm paid for consideration.

## Severity

| Severity | Examples |
|----------|---------|
| **HIGH** | Client quote with no compensation disclosure; cherry-picked testimonials only; misleading endorsement |
| **MEDIUM** | Generic "results vary" buried in a footnote; vague attribution; missing context about the relationship |
| **LOW** | Existing disclosures could be strengthened |

## Acceptable vs. Problematic Examples

✅ `"'Working with [Advisor] has helped our retirement planning,' says John S., client since 2020. Client was not compensated. Individual experiences may not be representative."`

❌ `"Best advisor ever!" – Client` — missing compensation disclosure and atypical results disclaimer

## Applicable Rules

- FINRA Rule 2210(d)(6)
- IA Act Rule 206(4)-1(b) — Investment Adviser Marketing Rule

## Safe-Fail

- Any client quote → Requires full disclosures (compensation + atypical results)
- Third-party rating or award → Requires attribution and criteria disclosure
- Uncertain whether the testimonial provider was compensated → Flag and require disclosure
