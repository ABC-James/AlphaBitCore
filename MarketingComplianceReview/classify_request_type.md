# Node: classify_request_type

| Property | Value |
|----------|-------|
| **ID** | `classify_request_type` |
| **Type** | `MODEL` |
| **Version** | `1.0.0` |
| **Position** | x: 100, y: 725 |

## Purpose

Entry point for the workflow. Determines whether the incoming message contains actual marketing/advertising content eligible for a compliance review. If yes, extracts the content and classifies key metadata. If no, triggers the fast-fail path.

## Execution Guard

Always executes — this is the root node.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{message}}` | The raw user message |

## Output Schema

Returns a JSON object:

```json
{
  "is_compliance_request": true | false,
  "rejection_reason": "specific reason" | null,
  "firm_type": "Broker-Dealer | RIA | Dual Registrant | Unknown",
  "audience": "Retail | Institutional | Internal",
  "channel": "Social | Email | Website | Pitchbook | Factsheet | Presentation | Other",
  "content_extracted": "actual marketing text",
  "content_length_chars": 0,
  "confidence": 0.0
}
```

## Decision Logic

| Condition | Result |
|-----------|--------|
| Contains marketing/advertising content for review | `is_compliance_request = true` |
| Question about rules/regulations only | `is_compliance_request = false` |
| "How do I write…" style question | `is_compliance_request = false` |
| "Is it okay to say…" without actual content | `is_compliance_request = false` |
| Empty message | `is_compliance_request = false`, rejection_reason = "No content provided" |

## Safe-Fail Rules

- If **any** marketing text is present → ACCEPT (err on the side of reviewing)
- If **only** questions with no content → REJECT
- `confidence < 0.7` when the classification is borderline

## Edge Cases

| Input | Behavior |
|-------|----------|
| `"Can I say guaranteed returns?"` | REJECT — question about rules, not content review |
| `"Review this: [content]"` | ACCEPT if content exists |
| `"Is this okay? [content]"` | ACCEPT — extract the implied content |

## Downstream Edges

| Target Node | Condition |
|-------------|-----------|
| `fast_fail_handler` | `is_compliance_request == false` |
| `classify_content_complexity` | `is_compliance_request == true` |
| `classify_asset_class` | `is_compliance_request == true` |
| `synthesize_results` | Always (passes classification metadata) |
| `format_output` | Always (passes through) |
