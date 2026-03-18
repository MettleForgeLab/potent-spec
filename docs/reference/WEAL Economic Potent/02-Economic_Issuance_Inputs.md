```markdown
Issuance Inputs — Structured Schema Specification  
Scope: Declarative input schema for rule-defined issuance calculation within the ledger protocol.  
Version: 1.1 (Revised)

---

## 1) Validated Event Record Types

### 1.1 Enumerated Event Categories

Event categories are descriptive classifications used to standardize input structure.  
They do not imply ranking, weighting, or qualitative assessment.

Allowed `event_type` values (enumerated):

- `INFORMATION_UPDATE`
- `DATA_SUBMISSION`
- `CORRECTION_NOTICE`
- `INFRASTRUCTURE_REPORT`
- `SERVICE_RECORD`
- `OBSERVATION_RECORD`
- `PROCESS_COMPLETION`
- `SYSTEM_SIGNAL`
- `RESOURCE_LOG`
- `INTERACTION_RECORD`

Additional event types may be defined in deployment-specific parameter sets, provided they are enumerated and documented.

---

### 1.2 Event Classification Schema

Each event MUST include:

- `event_type` (string, enumerated)
- `event_code` (string, deployment-defined code)
- `classification_version` (string)
- `schema_version` (string)

No free-form categorization is permitted outside defined enumerations.

---

### 1.3 Required Classification Identifiers

Each event record MUST contain:

- `event_id` (string; globally unique)
- `event_timestamp` (ISO-8601 UTC)
- `originating_institution_id` (string)
- `reporting_channel_id` (string; if applicable)

Identifiers must conform to canonical formatting defined in Section 5.

---

## 2) Required Data Fields

All issuance input records MUST contain the following structured fields:

- `event_id` (string)
- `event_timestamp` (ISO-8601 UTC)
- `originating_institution_id` (string)
- `verification_status` (enumerated)
- `scope_identifier` (enumerated)
- `quantitative_metric` (decimal; optional but schema-declared)
- `qualitative_code` (enumerated string; optional)
- `parameter_set_id` (string)
- `decay_profile_id` (string)

---

### 2.1 Verification Status Enumeration

Allowed values:

- `VERIFIED`
- `SELF_DECLARED`
- `THIRD_PARTY_CONFIRMED`
- `AUTOMATED_VALIDATION`
- `PENDING`

No value implies superiority or priority.

---

### 2.2 Quantitative Metric

If used, `quantitative_metric` MUST:

- Be numeric
- Declare unit in associated schema documentation
- Fall within bounds defined in the parameter set
- Be reproducible from source data

No implicit normalization is permitted.

---

### 2.3 Qualitative Code

If used, `qualitative_code` MUST:

- Be selected from a closed enumeration
- Be documented in the parameter set
- Not be interpreted outside declared mapping rules

Free-text qualitative input is not permitted in issuance calculation.

---

## 3) Scope Tiers (Non-Normative)

Scope identifiers describe structural reach only.

Allowed `scope_identifier` values:

- `LOCAL`
- `REGIONAL`
- `INTER_REGIONAL`

Additional scope tiers may be defined if enumerated.

These tiers describe scope of applicability only.  
They do not imply value hierarchy or priority.

---

## 4) Issuance Calculation Inputs

The issuance function receives a structured `inputs` object.

### 4.1 Required Structure
```

inputs = {
event_type: string,
event_id: string,
scope_identifier: string,
issuer_defined_weight: decimal,
federation_multiplier: decimal,
quantitative_metric: decimal (optional),
qualitative_code: string (optional),
verification_status: string,
parameter_set_id: string
}

```

---

### 4.2 Parameter Linkage

- `issuer_defined_weight` MUST be defined in the referenced parameter set.
- `federation_multiplier` MUST be defined in federation parameters.
- Bounds MUST be enforced prior to issuance calculation.
- Missing parameter references MUST cause validation failure.

---

### 4.3 No Discretionary Override

All issuance calculations:

- MUST use declared inputs only.
- MUST not incorporate undeclared fields.
- MUST not include manual adjustments.
- MUST be reproducible using stored input fields and parameter set.

---

## 5) Reproducibility Requirements

### 5.1 Canonical Field Formatting

All string fields MUST:

- Use UTF-8 encoding
- Be case-sensitive unless explicitly normalized
- Avoid trailing whitespace
- Follow declared naming conventions

Numeric fields MUST:

- Use fixed-point decimal representation where declared
- Apply declared rounding mode
- Maintain defined precision

---

### 5.2 Normalization Rules

Prior to issuance calculation:

- Timestamps MUST be normalized to UTC ISO-8601.
- Enumerated values MUST match declared enumeration exactly.
- Null or undefined optional fields MUST be explicitly set to `null`.

Implicit defaulting is not permitted unless defined in the parameter set.

---

### 5.3 Validation Preconditions

Issuance input record MUST:

- Pass schema validation
- Pass enumeration validation
- Pass bounds validation
- Reference existing `parameter_set_id`
- Reference existing `decay_profile_id`

No inference or derived scoring is permitted outside defined schema.

---

### 5.4 Reproducible Reconstruction

Given:

- A stored `inputs` object
- The referenced parameter set
- The defined issuance function

The issued amount MUST be reproducible without additional context.

---

End of File.
```
