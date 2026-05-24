# Governance Metadata Contracts

## Objective

Define standard metadata keys and conventions for MedMesh governance decisions.

The existing AgentGuard runtime already allows guards to return metadata through GuardResult.metadata.

MedMesh should use this metadata field carefully as the first extension mechanism for healthcare governance before introducing strongly typed metadata classes.

This document defines the initial metadata contract for:

- PHI detection
- PHI transformation
- policy decisions
- FHIR access control
- audit events
- clinical risk classification
- runtime enforcement
- observability

---

## Why This Matters

Healthcare AI systems require strong auditability.

A governance system must be able to answer:

- What was detected?
- What was modified?
- What was blocked?
- Which policy was applied?
- Which guard made the decision?
- Which workflow was affected?
- Was PHI present?
- Was patient context present?
- Was a FHIR resource accessed?
- Was a human review required?
- Was the decision enforced or only observed?

Without a metadata contract, each guard may return inconsistent metadata.

That would make auditability, observability, research evaluation, and compliance reporting difficult.

---

## Design Principles

### 1. Metadata Must Be Explicit

Every important governance decision should be represented clearly.

Avoid vague keys like:

- info
- data
- result
- details

Prefer explicit keys like:

- phi_detected
- policy_decision
- fhir_resource_type
- audit_required

---

### 2. Metadata Must Avoid Raw PHI

Guard metadata should not store raw patient identifiers.

Avoid storing:

- patient names
- phone numbers
- email addresses
- medical record numbers
- addresses
- raw clinical note snippets
- raw FHIR payloads

Use safer alternatives:

- entity type
- entity count
- confidence score
- hashed value
- transformation type
- policy ID
- audit event ID

---

### 3. Metadata Must Be Stable

Once a metadata key is used publicly, avoid changing its meaning.

If the meaning changes, introduce a new versioned key or policy version.

---

### 4. Metadata Must Support Auditability

Metadata should be sufficient to reconstruct governance behavior without exposing raw PHI.

---

### 5. Metadata Must Support Research Evaluation

MedMesh has a research and EB1A strategy.

Metadata should support:

- benchmarking
- false positive analysis
- false negative analysis
- latency analysis
- policy evaluation
- guard performance analysis

---

## Naming Convention

Use snake_case for metadata keys.

Examples:

- phi_detected
- entity_count
- policy_id
- audit_required
- fhir_resource_type

Do not use mixed styles like:

- phiDetected
- PHI_Detected
- PhiDetected

---

## Common Metadata Keys

These keys may appear across multiple guard types.

### guard_category

Purpose:

    Identifies the broad guard category.

Example values:

- structural
- pattern
- budget
- scope
- pii
- phi
- semantic
- audit
- fhir
- clinical_safety

---

### enforcement_mode

Purpose:

    Records the runtime mode of the guard.

Example values:

- enforce
- observe
- disabled

---

### policy_id

Purpose:

    Identifies the policy or rule applied by the guard.

Example:

    phi.safe_harbor.v1

---

### policy_version

Purpose:

    Identifies the policy version.

Example:

    1.0.0

---

### policy_decision

Purpose:

    Records the policy decision.

Example values:

- allow
- deny
- modify
- observe
- review_required

Note:

The current GuardVerdict values remain PASS, BLOCK, and MODIFY.

policy_decision can provide richer metadata without changing GuardVerdict yet.

---

### audit_required

Purpose:

    Indicates whether the event should be written to an audit trail.

Example values:

- true
- false

---

### risk_level

Purpose:

    Indicates the risk classification.

Example values:

- low
- medium
- high
- critical

---

### reason_code

Purpose:

    Machine-readable reason for a decision.

Example values:

- phi_detected
- unauthorized_tool
- fhir_scope_missing
- patient_context_missing
- unsafe_clinical_output
- budget_exceeded

---

### human_readable_reason

Purpose:

    A readable summary of why the decision was made.

Example:

    PHI was detected in tool output and external model policy requires masking.

---

## PHI Detection Metadata

These keys should be used by PHI-related guards.

### phi_detected

Type:

    boolean

Purpose:

    Indicates whether PHI was detected.

Example:

    true

---

### phi_types

Type:

    list of strings

Purpose:

    Lists detected PHI entity types.

Example:

    ["PERSON", "DATE", "MEDICAL_RECORD_NUMBER"]

---

### pii_types

Type:

    list of strings

Purpose:

    Lists generic PII entity types from the underlying detector.

Example:

    ["PERSON", "PHONE_NUMBER", "EMAIL_ADDRESS"]

---

### entity_count

Type:

    integer

Purpose:

    Total number of detected entities.

Example:

    5

---

### entity_summary

Type:

    object

Purpose:

    Count of detected entities by type.

Example:

    {
      "PERSON": 2,
      "DATE": 1,
      "PHONE_NUMBER": 1,
      "MEDICAL_RECORD_NUMBER": 1
    }

Important:

Do not include raw entity values.

---

### confidence_threshold

Type:

    number

Purpose:

    Detector confidence threshold used for the decision.

Example:

    0.7

---

### max_confidence_score

Type:

    number

Purpose:

    Highest detector confidence score observed.

Example:

    0.94

---

### detector_backend

Type:

    string

Purpose:

    Identifies the detection backend.

Example values:

- presidio
- regex
- custom_ner
- clinical_bert
- llm_judge

---

## PHI Transformation Metadata

These keys should be used when content is modified.

### handling_mode

Type:

    string

Purpose:

    Describes how PHI was handled.

Example values:

- detect_only
- mask
- redact
- pseudonymize
- anonymize
- semantic_replacement
- block

---

### anonymization_operator

Type:

    string

Purpose:

    Describes the transformation operator.

Example values:

- replace
- redact
- hash
- date_shift
- generalize
- suppress
- surrogate

---

### reversible

Type:

    boolean

Purpose:

    Indicates whether transformation can be reversed.

Example:

    false

---

### mapping_id

Type:

    string or null

Purpose:

    References a secure mapping record if pseudonymization is reversible.

Important:

Never store the mapping itself in metadata.

Example:

    map_01HR7K9Q2M

---

### original_value_hashes

Type:

    list of strings

Purpose:

    Stores salted hashes of original values when needed for audit or consistency checks.

Important:

Do not store raw values.

Example:

    ["sha256:..."]

---

### replacement_strategy

Type:

    string

Purpose:

    Indicates the replacement strategy.

Example values:

- placeholder
- consistent_token
- surrogate_name
- generalized_date
- age_band
- removed

---

## HIPAA Safe Harbor Metadata

These keys should be used when Safe Harbor-style handling is applied.

### safe_harbor_applied

Type:

    boolean

Purpose:

    Indicates whether Safe Harbor-style handling was applied.

Example:

    true

---

### safe_harbor_categories

Type:

    list of strings

Purpose:

    Lists Safe Harbor categories detected or handled.

Example values:

- names
- geographic_subdivisions
- dates
- telephone_numbers
- fax_numbers
- email_addresses
- social_security_numbers
- medical_record_numbers
- health_plan_beneficiary_numbers
- account_numbers
- certificate_license_numbers
- vehicle_identifiers
- device_identifiers
- urls
- ip_addresses
- biometric_identifiers
- full_face_images
- unique_identifying_codes

---

### residual_risk_level

Type:

    string

Purpose:

    Estimated residual risk after transformation.

Example values:

- very_low
- low
- medium
- high
- unknown

---

### deidentification_mode

Type:

    string

Purpose:

    Indicates the de-identification governance mode.

Example values:

- safe_harbor
- expert_determination
- pseudonymization
- internal_minimization

---

## FHIR Authorization Metadata

These keys should be used by FHIR-aware guards.

### fhir_access_requested

Type:

    boolean

Purpose:

    Indicates whether a FHIR operation was requested.

Example:

    true

---

### fhir_resource_type

Type:

    string

Purpose:

    FHIR resource type involved in the request.

Example values:

- Patient
- Observation
- MedicationRequest
- Condition
- Encounter
- DiagnosticReport
- AllergyIntolerance

---

### fhir_operation

Type:

    string

Purpose:

    FHIR operation requested.

Example values:

- read
- search
- create
- update
- delete
- patch

---

### fhir_scope_required

Type:

    string

Purpose:

    Required SMART on FHIR scope or internal equivalent.

Example:

    patient/Observation.read

---

### fhir_scope_present

Type:

    boolean

Purpose:

    Indicates whether the required scope was present.

Example:

    true

---

### patient_context_required

Type:

    boolean

Purpose:

    Indicates whether patient context was required.

Example:

    true

---

### patient_context_present

Type:

    boolean

Purpose:

    Indicates whether patient context was present.

Example:

    true

---

### patient_scope_match

Type:

    boolean

Purpose:

    Indicates whether requested patient context matches authorized patient context.

Example:

    true

---

### data_sensitivity

Type:

    string

Purpose:

    Classifies the sensitivity of the requested data.

Example values:

- normal
- sensitive
- restricted
- highly_restricted

---

## Tool Governance Metadata

These keys apply to general tool authorization and future healthcare tool governance.

### tool_name

Type:

    string

Purpose:

    Name of the requested tool.

Example:

    fhir_read_observation

---

### tool_allowed

Type:

    boolean

Purpose:

    Whether the tool is allowed in the current context.

Example:

    true

---

### allowed_tools_snapshot

Type:

    list of strings

Purpose:

    Optional snapshot of allowed tools at decision time.

Important:

Use only when helpful and not too noisy.

---

### tool_risk_level

Type:

    string

Purpose:

    Risk level assigned to the tool.

Example values:

- low
- medium
- high
- critical

---

### tool_action_type

Type:

    string

Purpose:

    Type of action the tool performs.

Example values:

- read
- write
- delete
- export
- notify
- summarize
- classify

---

### bulk_access_detected

Type:

    boolean

Purpose:

    Indicates whether the request appears to involve bulk access.

Example:

    false

---

## Clinical Safety Metadata

These keys should be used by clinical output and semantic safety guards.

### clinical_safety_checked

Type:

    boolean

Purpose:

    Indicates whether clinical safety was evaluated.

Example:

    true

---

### clinical_risk_level

Type:

    string

Purpose:

    Risk classification of the clinical output or request.

Example values:

- low
- medium
- high
- critical

---

### unsafe_clinical_advice_detected

Type:

    boolean

Purpose:

    Indicates whether unsafe clinical advice was detected.

Example:

    false

---

### requires_human_review

Type:

    boolean

Purpose:

    Indicates whether human review is required.

Example:

    true

---

### review_reason

Type:

    string

Purpose:

    Reason human review is required.

Example:

    medication_change_recommendation

---

### patient_facing_output

Type:

    boolean

Purpose:

    Indicates whether output is intended for a patient.

Example:

    true

---

### clinician_facing_output

Type:

    boolean

Purpose:

    Indicates whether output is intended for a clinician.

Example:

    false

---

## Audit Metadata

These keys should be used by audit and monitoring guards.

### audit_event_id

Type:

    string

Purpose:

    Unique ID for the audit event.

Example:

    audit_01HR7K9Q2M

---

### audit_event_type

Type:

    string

Purpose:

    Type of audit event.

Example values:

- phi_detected
- phi_transformed
- policy_allowed
- policy_blocked
- fhir_access_allowed
- fhir_access_denied
- output_blocked
- human_review_required

---

### audit_sink

Type:

    string

Purpose:

    Where the audit event was written.

Example values:

- stdout
- file
- otel
- database
- siem
- disabled

---

### trace_id

Type:

    string

Purpose:

    Distributed trace ID.

Example:

    4bf92f3577b34da6a3ce929d0e0e4736

---

### span_id

Type:

    string

Purpose:

    Span ID for the relevant operation.

Example:

    00f067aa0ba902b7

---

### session_id

Type:

    string

Purpose:

    Session identifier from AgentContext.

Example:

    session_123

---

### task_id

Type:

    string

Purpose:

    Task identifier from AgentContext.

Example:

    task_456

---

## Budget and Rate Metadata

These keys should be used by budget guards.

### budget_type

Type:

    string

Purpose:

    Type of budget checked.

Example values:

- tokens
- cost
- requests
- tool_calls

---

### budget_remaining

Type:

    number

Purpose:

    Remaining budget before or after decision.

Example:

    12000

---

### budget_required

Type:

    number

Purpose:

    Budget required for the requested action.

Example:

    5000

---

### budget_exceeded

Type:

    boolean

Purpose:

    Indicates whether a budget was exceeded.

Example:

    false

---

## Metadata Examples

### Example 1: PHI Detected and Masked

    {
      "guard_category": "phi",
      "enforcement_mode": "enforce",
      "phi_detected": true,
      "phi_types": ["PERSON", "DATE", "MEDICAL_RECORD_NUMBER"],
      "entity_count": 3,
      "handling_mode": "mask",
      "anonymization_operator": "replace",
      "safe_harbor_applied": true,
      "safe_harbor_categories": ["names", "dates", "medical_record_numbers"],
      "policy_id": "phi.safe_harbor.v1",
      "policy_version": "1.0.0",
      "reversible": false,
      "audit_required": true,
      "risk_level": "medium"
    }

---

### Example 2: FHIR Access Allowed

    {
      "guard_category": "fhir",
      "enforcement_mode": "enforce",
      "policy_id": "fhir.patient_read.v1",
      "policy_decision": "allow",
      "fhir_access_requested": true,
      "fhir_resource_type": "Observation",
      "fhir_operation": "read",
      "fhir_scope_required": "patient/Observation.read",
      "fhir_scope_present": true,
      "patient_context_required": true,
      "patient_context_present": true,
      "patient_scope_match": true,
      "audit_required": true,
      "risk_level": "low"
    }

---

### Example 3: Tool Access Blocked

    {
      "guard_category": "scope",
      "enforcement_mode": "enforce",
      "policy_id": "tool.scope.v1",
      "policy_decision": "deny",
      "tool_name": "bulk_patient_export",
      "tool_allowed": false,
      "tool_risk_level": "critical",
      "bulk_access_detected": true,
      "reason_code": "unauthorized_tool",
      "human_readable_reason": "The agent is not allowed to invoke bulk patient export.",
      "audit_required": true,
      "risk_level": "critical"
    }

---

### Example 4: Clinical Output Requires Review

    {
      "guard_category": "clinical_safety",
      "enforcement_mode": "observe",
      "clinical_safety_checked": true,
      "clinical_risk_level": "high",
      "unsafe_clinical_advice_detected": false,
      "requires_human_review": true,
      "review_reason": "medication_change_recommendation",
      "patient_facing_output": true,
      "policy_decision": "review_required",
      "audit_required": true
    }

---

## Initial Metadata Contract Version

Current metadata contract version:

    medmesh.governance_metadata.v0.1

This version is intentionally lightweight.

It is designed to work with the existing GuardResult.metadata field without requiring immediate runtime refactoring.

---

## Rules For New Guards

Any new MedMesh guard should:

1. Use documented metadata keys where possible.
2. Avoid storing raw PHI.
3. Include policy_id when a policy decision is made.
4. Include audit_required when the decision has compliance relevance.
5. Include reason_code for BLOCK decisions.
6. Include handling_mode for PHI transformations.
7. Include fhir_resource_type and fhir_operation for FHIR decisions.
8. Include risk_level when risk classification is possible.
9. Keep metadata concise enough for logging and tracing.
10. Document any new metadata keys before using them widely.

---

## Keys To Avoid

Avoid vague or unsafe metadata keys:

- raw_text
- raw_phi
- original_value
- patient_name
- full_note
- full_payload
- secret
- token
- password
- data
- details

If sensitive values are needed for consistency checks, store salted hashes or secure mapping references instead.

---

## Future Typed Metadata Classes

The current metadata contract uses plain dictionaries.

Later, MedMesh may introduce typed metadata classes such as:

- PHIMetadata
- FHIRAuthorizationMetadata
- PolicyDecisionMetadata
- ClinicalSafetyMetadata
- AuditMetadata
- BudgetMetadata

This should happen only after the metadata patterns stabilize.

---

## Principal Engineer Recommendation

Use GuardResult.metadata as the initial healthcare governance extension seam.

Do not introduce complex typed metadata too early.

Instead:

1. standardize metadata keys now
2. implement one or two healthcare guards
3. observe real metadata needs
4. then introduce typed metadata classes if needed

This keeps the system simple while preserving an upgrade path.

---

## Next Recommended Document

Create:

    docs/architecture/guard-implementation-analysis-correction.md

Purpose:

    Correct earlier guard implementation documentation so it matches the actual verified file structure.

This is important because we previously used conceptual filenames that do not match the current repository layout.

