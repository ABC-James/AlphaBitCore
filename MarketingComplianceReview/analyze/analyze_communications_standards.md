# Node: analyze_communications_standards

| Property | Value |
|----------|-------|
| **ID** | `analyze_communications_standards` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3000` |
| **Position** | x: 800, y: 950 |

## Purpose

Assesses whether the content meets FINRA Rule 2210's general communications standards: fair and balanced presentation, non-misleading claims, sound basis for factual assertions, and absence of material omissions. Applies standards proportionally based on content type — brand copy is treated differently from investment performance claims.

## Execution Guard

Executes **only when** `{{classify_request_type.is_compliance_request}} == true`.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.firm_type}}` | Firm registration type |
| `{{classify_request_type.audience}}` | Retail, Institutional, or Internal |
| `{{classify_request_type.channel}}` | Communication channel |
| `{{classify_request_type.content_extracted}}` | The marketing content |

## Output Schema

```json
{
  "issues": [
    {
      "severity": "HIGH | MEDIUM | LOW",
      "issue": "specific issue",
      "rule_violated": "rule",
      "explanation": "why problematic",
      "suggested_revision": "how to fix"
    }
  ],
  "has_issues": true | false,
  "highest_severity": "HIGH | MEDIUM | LOW | NONE",
  "is_balanced": true | false,
  "has_material_omissions": true | false
}
```

## Content Type Classification

Before applying standards, content is categorized into one of four types:

| Type | Description | Examples |
|------|-------------|---------|
| **A — Brand/Firm Marketing** | Taglines, mission statements, service descriptions | "Empowering investors since 1985", "World-class service" |
| **B — Product/Strategy Descriptions** | What the fund/strategy does | "The fund invests in large-cap US equities" |
| **C — Performance and Outcome Claims** | Specific returns, rankings, comparisons | "Returned 15% in 2024", "Top quartile since inception" |
| **D — Investment Recommendations** | Specific calls to action or advice | "Now is the time to invest in equities" |

Standards are applied based on type — Type A and B content is held to a much lower bar than Type C and D.

## Core Standards

### 1. Fair and Balanced (FINRA 2210(d)(1)(A))
Applies to **Type C and Type D only**.

✅ Brand copy (Type A) does NOT require risk disclosures paired with every positive statement.

✅ `"Fund returned 18% in 2024 (net), vs S&P 500 11.2%. Past performance does not guarantee future results. Investing involves risk of loss."` — balanced

❌ `"18% return in 2024!"` with no disclaimer and no risk context — unbalanced (HIGH for Type C)

### 2. Not Misleading (FINRA 2210(d)(1)(B))
Cannot: omit material facts, make factually false statements, use misleading charts, imply non-existent guarantees, or misrepresent product features.

Enthusiastic but non-specific language ("exceptional", "innovative", "leading") is NOT misleading — it is normal commercial speech.

### 3. Sound Basis (FINRA 2210(d)(1)(C))
Applies to specific, verifiable factual claims only.

✅ `"90% client retention rate (internal data, 2024)"` — needs data to back it up
✅ `"Top quartile Sharpe ratio per Morningstar"` — needs source citation
✅ `"Trusted by clients for decades"` — general puffery, no substantiation required

### 4. Material Omissions
Applies when specific investment facts are presented selectively to create a false impression. Does **not** apply to brand positioning copy.

## Channel-Specific Rules

| Channel | Requirement |
|---------|-------------|
| Social Media | If specific claims made, disclosures must link out; brand copy needs no disclosure |
| Email | Subject line cannot be factually misleading; promotional subject lines are fine |
| Website | Performance disclosures on same page as data; brand sections need no disclaimers |
| Pitchbooks | Each slide with performance data must stand alone fairly; brand/intro slides are fine |

## Audience Rules

| Audience | Requirement |
|----------|-------------|
| Retail | Investment claims must be understandable to average investor; plain English required |
| Institutional | Technical language acceptable throughout; specific claims must still be fair |

## Common Issue Calibration

| Issue Type | Severity |
|-----------|---------|
| Specific returns with no risk context or disclaimer | HIGH |
| Product description with outcome claims but no risk mention | MEDIUM |
| Brand copy emphasizing positives | NON-ISSUE |
| "Strong performance" as standalone phrase | LOW or NON-ISSUE |
| Returns without time period or net/gross designation | MEDIUM |
| Promotional enthusiasm | NON-ISSUE |

## Applicable Rules

- FINRA Rule 2210(d)(1) — Content Standards
- SEC Rule 10b-5 — Anti-Fraud
- SEC Rule 156

## Safe-Fail

- Specific investment performance/outcome claims without risk context → Flag as imbalanced
- Brand/service copy without investment claims → Do NOT flag as imbalanced
- Verifiable factual claims without sourcing → Flag for substantiation
- Aspirational or puffery language → Do NOT flag
- General promotional enthusiasm → Do NOT flag
