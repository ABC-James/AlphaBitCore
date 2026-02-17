# Node: classify_content_complexity

| Property | Value |
|----------|-------|
| **ID** | `classify_content_complexity` |
| **Type** | `MODEL` |
| **Version** | `1.0.0` |
| **Position** | x: 450, y: 575 |

## Purpose

Analyzes the extracted marketing content to measure its length and identify what types of claims it contains. The output drives which downstream analyzers are invoked, optimizing execution by skipping analyzers that aren't relevant.

## Execution Guard

Executes **only when** `{{classify_request_type.is_compliance_request}} == true`.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.content_extracted}}` | The extracted marketing text |
| `{{classify_request_type.channel}}` | Communication channel |
| `{{classify_request_type.content_length_chars}}` | Character count |

## Output Schema

```json
{
  "complexity": "MINIMAL | LIGHT | MODERATE | COMPREHENSIVE",
  "word_count": 0,
  "sentence_count": 0,
  "has_performance_claims": true | false,
  "has_numerical_claims": true | false,
  "has_testimonials": true | false,
  "has_endorsements": true | false,
  "has_risk_language": true | false,
  "has_rankings": true | false,
  "has_comparisons": true | false,
  "requires_performance_analysis": true | false,
  "requires_testimonial_analysis": true | false,
  "requires_risk_analysis": true | false,
  "requires_recordkeeping_analysis": true | false,
  "reasoning": "brief explanation"
}
```

## Complexity Levels

| Level | Criteria |
|-------|----------|
| `MINIMAL` | 1–50 words, social media, no claims |
| `LIGHT` | 51–150 words, basic claims |
| `MODERATE` | 151–400 words, multiple claim types |
| `COMPREHENSIVE` | 400+ words, complex |

## Claim Detection Rules

**`has_performance_claims`** — Set `true` for specific historical or projected returns with numbers (e.g., "returned 12%"). Do **not** set for puffery like "strong performance" or "attractive risk-adjusted returns."

**`has_numerical_claims`** — Set `true` for specific percentages or dollar amounts tied to investment outcomes. Do **not** set for general firm statistics like "$2B AUM" or "30 years in business."

**`has_testimonials`** — Client quotes attributed to a specific named or identified person.

**`has_rankings`** — Explicit quantified rankings like "#1" or "top quartile." Do **not** set for unquantified superlatives like "best-in-class."

**`has_comparisons`** — Explicit "vs" or "compared to" with specific benchmark or competitor data including actual numbers.

## Requires Flags Logic

| Flag | Triggers When |
|------|---------------|
| `requires_performance_analysis` | `has_performance_claims == true` OR `has_numerical_claims == true` OR `complexity >= MODERATE` |
| `requires_testimonial_analysis` | `has_testimonials == true` OR `complexity == COMPREHENSIVE` |
| `requires_risk_analysis` | `word_count > 150` OR `complexity >= MODERATE` |
| `requires_recordkeeping_analysis` | `word_count > 200` OR `complexity >= MODERATE` |

## Safe-Fail Rules

- Uncertain about a claim → Ask: does it contain a specific verifiable number tied to investment outcomes? If yes, flag it. If aspirational or descriptive without a verifiable number, do not flag.
- Borderline complexity → Choose the higher level.

## Downstream Edges

Feeds `requires_*` flags to all Stage 3 analyzer nodes.
