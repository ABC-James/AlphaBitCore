# Node: synthesize_results

| Property | Value |
|----------|-------|
| **ID** | `synthesize_results` |
| **Type** | `TOOL` |
| **Tool** | `OpenAI_chat` |
| **Model** | `gpt-4o` |
| **Temperature** | `0.4` |
| **Max Tokens** | `4000` |
| **Position** | x: 1150, y: 725 |

## Purpose

Aggregates the outputs from all upstream classification and analyzer nodes into a single, coherent compliance verdict. Applies a balanced Chief Compliance Officer (CCO) perspective — protecting investors from genuine regulatory violations while avoiding unnecessary friction with standard marketing practices.

## Execution Guard

Always executes (runs after all analyzers complete).

## Inputs

| Variable | Source Node |
|----------|-------------|
| `{{classify_request_type}}` | Request classifier |
| `{{classify_content_complexity}}` | Content complexity classifier |
| `{{classify_asset_class}}` | Asset class classifier |
| `{{analyze_prohibited_statements}}` | Prohibited statements analyzer |
| `{{analyze_performance_claims}}` | Performance claims analyzer |
| `{{analyze_risk_disclosures}}` | Risk disclosure analyzer |
| `{{analyze_testimonials}}` | Testimonials analyzer |
| `{{analyze_communications_standards}}` | Communications standards analyzer |
| `{{analyze_recordkeeping}}` | Recordkeeping analyzer |
| `{{analyze_hedge_fund_disclosures}}` | Hedge fund analyzer |
| `{{analyze_private_placement_disclosures}}` | Private placement analyzer |

## Output Schema

```json
{
  "overall_risk": "LOW | MEDIUM | HIGH | CRITICAL",
  "status": "ACCEPT | NEEDS CHANGES | REJECT | ESCALATION REQUIRED",
  "high_severity_issues": ["issue1", "issue2"],
  "medium_severity_issues": ["issue1"],
  "low_severity_issues": ["issue1"],
  "required_edits": [
    {
      "edit": "specific change",
      "rationale": "why",
      "rule": "FINRA/SEC rule"
    }
  ],
  "required_disclosures": "Full block of disclosure text (copy-paste ready)",
  "substantiation_needed": ["claim requiring proof"],
  "escalation_required": true | false,
  "escalation_reason": "reason or null",
  "recordkeeping_metadata": {},
  "analyzers_run": ["list of which analyzers executed"],
  "complexity_classification": "from complexity analyzer",
  "asset_complexity": "from asset classifier"
}
```

## Overall Risk Determination

| Level | Conditions |
|-------|-----------|
| **CRITICAL** | Prohibited statements (guarantees, risk-free claims); missing VERY HIGH asset disclosures for retail; performance with zero disclaimers; potential fraud; 3+ HIGH issues |
| **HIGH** | 1–2 HIGH issues; HIGH asset without adequate disclosure; material performance violations; testimonial violations |
| **MEDIUM** | 1+ MEDIUM issues; incomplete required disclosures; 3+ LOW issues that collectively create a misleading impression |
| **LOW** | Only LOW issues; minor improvements suggested — content is acceptable to publish |

> **Important:** Multiple LOW issues alone do **not** automatically escalate to MEDIUM. LOW issues are advisory and do not block approval unless they collectively create a materially misleading impression.

## Status Determination

| Status | Conditions |
|--------|-----------|
| **ACCEPT** | `overall_risk = LOW`; no HIGH or MEDIUM issues; disclosures adequate for content type |
| **NEEDS CHANGES** | `overall_risk = MEDIUM` or isolated HIGH issues fixable with targeted edits |
| **REJECT** | `overall_risk = HIGH` with multiple HIGH issues; requires substantial rework |
| **ESCALATION REQUIRED** | `overall_risk = CRITICAL`; prohibited statements; potential fraud; regulatory filing at risk; legal review needed |

## Escalation Triggers

- Prohibited language specifically about investment returns (guarantees, risk-free claims)
- Missing accredited investor disclosure for VERY HIGH assets marketed to retail
- Unregistered offering without proper Reg D disclosures
- Performance presented with zero disclaimers and no risk context whatsoever
- Fraudulent or materially false factual claims
- Testimonial violations likely to trigger enforcement action

## Required Edits

Generated for each HIGH and MEDIUM issue:
```json
{
  "edit": "Remove 'guaranteed returns' language",
  "rationale": "Guarantees prohibited under FINRA 2210",
  "rule": "FINRA Rule 2210(d)(1)(B)"
}
```

LOW issues are **not** included in `required_edits` — they appear as optional improvements in `format_output`.

## Required Disclosures

Compiled into a single copy-paste ready block. Consolidated to avoid redundancy. Only disclosures actually triggered by the content are included — no speculative additions.

## Substantiation Needed

Lists only specific verifiable claims requiring documentary proof:
- ✅ "Best performing fund in category" → Needs Morningstar/Lipper data
- ✅ "90% client retention" → Needs internal records
- ❌ "Exceptional service" → General puffery, no substantiation needed

## Balanced CCO Perspective

The goal is to protect investors and ensure regulatory compliance — not to sanitize all marketing enthusiasm. Standard marketing language, puffery, and aspirational brand copy serve legitimate business purposes. Reserve HIGH severity and REJECT/ESCALATION for genuine regulatory violations.

## Applicable Rules

All rules from upstream analyzers.

## Safe-Fail

- Any analyzer flagged HIGH → Minimum `overall_risk = MEDIUM` (evaluate whether the HIGH flag was appropriate before escalating further)
- Prohibited investment return statements → `ESCALATION REQUIRED`
- Multiple LOW issues from marketing puffery → Do NOT escalate; remain `LOW`
- VERY HIGH assets to retail without disclosures → `CRITICAL` risk
- Uncertain about severity → Default to the less restrictive interpretation for borderline cases
