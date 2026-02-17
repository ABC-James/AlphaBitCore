# Node: format_output

| Property | Value |
|----------|-------|
| **ID** | `format_output` |
| **Type** | `MODEL` |
| **Version** | `1.0.0` |
| **Position** | x: 1500, y: 725 |

## Purpose

The final node in the workflow. Takes all upstream outputs and formats them into a single, valid JSON response. This node has no analytical responsibility — it only shapes and structures the data for the consumer of the API.

## Execution Guard

Always executes — this is the terminal node.

## Inputs

| Variable | Source Node |
|----------|-------------|
| `{{classify_request_type}}` | Request classifier |
| `{{classify_content_complexity}}` | Content complexity classifier |
| `{{classify_asset_class}}` | Asset class classifier |
| `{{synthesize_results}}` | Results synthesizer |
| `{{fast_fail_handler}}` | Fast fail handler (if triggered) |

## Output: Discriminated Union

The response is one of exactly two schemas, selected based on whether `{{fast_fail_handler}}` has content.

---

### Schema A — Rejected Request

Used when the request was not a compliance review.

```json
{
  "review_type": "REJECTED_REQUEST",
  "reason": "from classify_request_type.rejection_reason",
  "message": "from fast_fail_handler"
}
```

---

### Schema B — Compliance Review

Used for all actual compliance reviews.

```json
{
  "review_type": "COMPLIANCE_REVIEW",
  "summary": {
    "overall_risk": "LOW | MEDIUM | HIGH | CRITICAL",
    "status": "ACCEPT | NEEDS CHANGES | REJECT | ESCALATION REQUIRED",
    "escalation_required": true | false,
    "escalation_reason": "string or null"
  },
  "classification": {
    "complexity": "MINIMAL | LIGHT | MODERATE | COMPREHENSIVE | null",
    "asset_classes": ["string"],
    "highest_asset_complexity": "MINIMAL | LOW | MODERATE | HIGH | VERY_HIGH | null",
    "firm_type": "string or null",
    "audience": "Retail | Institutional | Internal | null",
    "channel": "string or null",
    "word_count": 0,
    "analyzers_run": ["string"]
  },
  "issues": {
    "high": [
      {
        "issue": "string",
        "rule_violated": "string",
        "explanation": "string",
        "suggested_fix": "string"
      }
    ],
    "medium": [...],
    "low": [...]
  },
  "issue_counts": {
    "high": 0,
    "medium": 0,
    "low": 0,
    "total": 0
  },
  "required_changes": [
    {
      "edit": "string",
      "rationale": "string",
      "rule": "string"
    }
  ],
  "optional_improvements": [
    {
      "suggestion": "string",
      "rationale": "string"
    }
  ],
  "required_disclosures": "string or null",
  "substantiation_needed": ["string"],
  "recordkeeping": {
    "communication_type": "string or null",
    "requires_principal_approval": true | false | null,
    "requires_pre_use_filing": true | false | null,
    "retention_period_years": 3 | null,
    "special_requirements": ["string"],
    "filing_category": "string or null"
  },
  "metrics": {
    "content_complexity": "string or null",
    "asset_complexity": "string or null",
    "analyzers_executed": 0,
    "analyzers_total": 8,
    "processing_version": "V4_BALANCED"
  },
  "disclaimer": "This AI-assisted compliance review provides guidance based on FINRA and SEC regulations. It does not constitute legal advice. Final approval should be made by a qualified compliance professional. Material changes require re-review."
}
```

## Population Rules

| Field | Rule |
|-------|------|
| `issues.high/medium/low` | Consolidate from all analyzers; deduplicate if same issue flagged by multiple analyzers |
| `required_changes` | Only HIGH and MEDIUM issues generate entries here |
| `optional_improvements` | LOW issues only; empty array if none |
| `required_disclosures` | Copy the full ready-to-paste block from `synthesize_results`; null if none required |
| `substantiation_needed` | Specific verifiable claims only; omit general marketing language; empty array if none |
| `recordkeeping` | Populate from `analyze_recordkeeping` if it ran; null all fields if it did not |
| `metrics.analyzers_executed` | Count of analyzer nodes that actually ran per `synthesize_results.analyzers_run` |

## Safe-Fail

- If any upstream node is missing or empty → Populate that field as `null` or an empty array. **Never omit a key from the schema.**
- If synthesis is missing but classification exists → Return Schema B with all issue arrays empty and summary fields null
- Never output any text outside the JSON object
