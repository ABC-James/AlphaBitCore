# Node: fast_fail_handler

| Property | Value |
|----------|-------|
| **ID** | `fast_fail_handler` |
| **Type** | `MODEL` |
| **Version** | `1.0.0` |
| **Position** | x: 450, y: 725 |

## Purpose

Generates a polite, informative rejection message when the incoming request does not qualify as a marketing compliance review. Explains why the request was rejected and guides the user toward a valid submission.

## Execution Guard

Executes **only when** `{{classify_request_type.is_compliance_request}} == false`.

## Inputs

| Variable | Description |
|----------|-------------|
| `{{classify_request_type.rejection_reason}}` | Reason returned by the classifier |

## Output

A natural language response explaining the rejection. Uses the following template:

> I appreciate your interest in compliance review, but [explain why this doesn't qualify].
>
> This service is designed to review actual marketing/advertising content for FINRA and SEC compliance.
>
> If you have specific marketing materials you'd like reviewed (emails, social posts, factsheets, presentations), please share the content and I'll provide a detailed compliance analysis.

## Downstream Edges

| Target Node | Notes |
|-------------|-------|
| `format_output` | Passes the rejection message for final JSON wrapping (`review_type: "REJECTED_REQUEST"`) |
