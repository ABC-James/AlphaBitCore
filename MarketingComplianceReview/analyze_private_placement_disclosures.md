# Node: analyze_private_placement_disclosures

| Property | Value |
|----------|-------|
| **ID** | `analyze_private_placement_disclosures` |
| **Type** | `TOOL` |
| **Tool** | `Claude_chat` |
| **Model** | `claude-sonnet-4-5-20250929` |
| **Temperature** | `0.5` |
| **Max Tokens** | `3000` |
| **Position** | x: 800, y: 1250 |

## Purpose

Validates that marketing content for Regulation D private placements includes all required disclosures under federal securities laws. This is a specialized analyzer that only runs when the asset classification step identifies private placement characteristics.

## Execution Guard

Executes **only when**:
- `{{classify_request_type.is_compliance_request}} == true` **AND**
- `{{classify_asset_class.run_private_placement_analyzer}} == true`

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

### 1. Unregistered Offering Notice (HIGH)
Must state:
> *"These securities have not been registered with the SEC"* or *"This is an unregistered offering."*

### 2. Accredited Investor Requirement (HIGH)
Must disclose investor eligibility requirements:
- **Rule 506(b):** Up to 35 non-accredited sophisticated investors allowed; no general solicitation
- **Rule 506(c):** Accredited investors only; general solicitation permitted but accreditation must be **verified**

Example: *"This offering is conducted under Rule 506(c) of Regulation D and is available only to verified accredited investors."*

### 3. Transfer Restrictions (HIGH)
Must warn:
> *"Securities are restricted and may not be transferred except in compliance with securities laws."*

Example: *"These securities are subject to substantial restrictions on transfer and may not be sold except in compliance with Rule 144 or other exemptions."*

### 4. Risk of Total Loss (HIGH)
Must include:
> *"Investors may lose their entire investment"* or *"This involves a high degree of risk."*

### 5. Illiquidity (HIGH)
Must disclose: no public market exists, securities are illiquid, no guarantee of a liquidity event.

Example: *"There is no public market for these securities and none is expected to develop. The securities are illiquid."*

### 6. Limited Information Availability (MEDIUM)
Should note: less information available vs. registered offerings, offering materials have not been reviewed by the SEC.

Example: *"The offering has not been reviewed by the SEC. Investors will have access to limited information compared to registered securities."*

### 7. Suitability (MEDIUM)
Should include: investment is suitable only for investors who can afford to lose the entire amount; requires financial sophistication.

## Reg D Rule Specifics

| Rule | Key Requirements |
|------|-----------------|
| **506(b)** | Up to 35 non-accredited (but sophisticated) investors; no general solicitation |
| **506(c)** | Accredited investors only; general solicitation allowed; firm MUST verify accreditation |

## Complete Required Disclosure Example

> *"XYZ Fund is offering securities in a private placement under Rule 506(c) of Regulation D. These securities are not registered with the SEC and are available only to verified accredited investors. This is a speculative investment involving a high degree of risk, including risk of total loss of invested capital. There is no public market for the securities and transfer is subject to substantial restrictions under federal and state securities laws. Investors will have access to limited information compared to SEC-registered offerings. This investment is suitable only for sophisticated investors who can afford to lose their entire investment."*

## Applicable Rules

- Securities Act Rule 506
- Regulation D
- Rule 144 (resale exemptions)

## Safe-Fail

- Any Reg D mention → Verify unregistered offering disclosure is present
- Retail audience + private placement → Require accredited investor language (HIGH if missing)
- If 506(c) → Must specifically mention the verification requirement
