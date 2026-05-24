# MVP Scope and Demo Workflow

## Objective

Define the first MedMesh MVP clearly enough that implementation can begin without losing architectural focus.

This document answers:

- what the MVP is
- what the MVP is not
- which pipeline profile we implement first
- which guards are required
- what the first demo workflow should prove
- what success looks like
- what should be deferred

---

## MVP Thesis

The first MedMesh MVP should prove that healthcare AI agents can be governed at runtime before sensitive clinical data reaches an LLM or external model.

The MVP should focus on:

    PHI-safe clinical note summarization with external model protection.

This is intentionally narrow.

The goal is not to build a complete healthcare AI platform immediately.

The goal is to prove the core runtime value:

    detect, transform, govern, inspect, and audit healthcare AI execution.

---

## Selected MVP Profile

The first implemented profile should be:

    medmesh-external-model-strict

## Why This Profile First

This profile has the clearest safety story:

- raw PHI must not leave the trusted boundary
- prompts must be inspected before LLM use
- tool outputs must be inspected before model context
- final output must be checked for PHI leakage
- governance decisions must be auditable

This makes it ideal for:

- demo
- documentation
- research framing
- open-source clarity
- early healthcare AI governance validation

---

## MVP Demo Name

Recommended demo name:

    Secure Clinical Note Summarization With External Model Protection

Alternative names:

- PHI-Safe Clinical Summarization
- Governed Healthcare AI Note Summarization
- External LLM PHI Boundary Demo
- Runtime PHI Governance Demo

Preferred:

    PHI-Safe Clinical Summarization

---

## MVP User Story

As a healthcare AI developer,

I want MedMesh to inspect and transform clinical text before it is sent to an external LLM,

so that I can safely use AI summarization without leaking raw PHI.

---

## MVP Workflow

The first demo workflow should follow this path:

1. User submits a clinical note containing PHI.
2. MedMesh validates the input structure.
3. MedMesh detects PHI.
4. MedMesh masks or anonymizes the PHI.
5. MedMesh confirms that no raw PHI is sent to the external model.
6. The external model receives only the protected note.
7. The model generates a summary.
8. MedMesh inspects the output for PHI leakage.
9. MedMesh returns the safe summary.
10. MedMesh records governance metadata and audit events.

---

## MVP Execution Flow

### Stage 1: INPUT

Purpose:

    Validate and inspect user-provided clinical text.

Guards:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard
- PHIScrubberGuard

Expected behavior:

- invalid input is blocked
- prompt injection attempts are blocked
- detected PHI is transformed
- PHI metadata is attached

---

### Stage 2: PRE_LLM

Purpose:

    Ensure the model prompt is safe before the external LLM call.

Guards:

- PHIPromptBoundaryGuard
- ExternalModelPHIBlockGuard
- ModelDestinationPolicyGuard

Expected behavior:

- raw PHI is not allowed to leave the trusted boundary
- external model calls are blocked if PHI remains
- model destination policy is enforced

Note:

These guards may not exist yet.

They represent MVP implementation targets.

---

### Stage 3: LLM Call

Purpose:

    Send only protected clinical text to the model.

Expected behavior:

- external model receives masked or anonymized text
- model does not receive raw patient identifiers
- summary is generated from protected content

Out of scope:

- model fine-tuning
- model hosting
- model evaluation
- clinical reasoning validation

---

### Stage 4: OUTPUT

Purpose:

    Inspect the generated summary before returning it.

Guards:

- PHILeakageOutputGuard
- OutputPolicyGuard

Expected behavior:

- output is checked for PHI leakage
- unsafe leakage is blocked or redacted
- safe output is returned

---

### Stage 5: ASYNC

Purpose:

    Record governance and audit events.

Guards:

- HealthcareAuditGuard
- PHIAuditGuard
- PolicyDecisionAuditGuard

Expected behavior:

- detected PHI categories are logged
- transformation mode is logged
- policy decisions are logged
- raw PHI is not stored in audit metadata
- traceable runtime evidence is produced

---

## MVP In Scope

The MVP includes:

### 1. PHI Detection

Detect basic PHI categories using the existing Presidio foundation.

Initial categories may include:

- names
- phone numbers
- email addresses
- dates
- locations
- medical record-like identifiers where possible

---

### 2. PHI Transformation

Support at least one transformation mode:

    mask

Example:

    John Smith was admitted on March 3, 2024.

Becomes:

    [PERSON] was admitted on [DATE].

---

### 3. INPUT Inspection

Inspect user input before the LLM sees it.

---

### 4. PRE_LLM Boundary Check

Confirm that raw PHI is not present before external model invocation.

---

### 5. OUTPUT Inspection

Inspect final model output for PHI leakage.

---

### 6. Governance Metadata

Return metadata using the governance metadata contract.

Required metadata:

- phi_detected
- phi_types
- entity_count
- handling_mode
- policy_id
- policy_decision
- audit_required
- risk_level

---

### 7. Basic Audit Event

Write or emit an audit event.

Initial audit sink can be:

- console
- local JSON file
- structured log

OpenTelemetry can come later.

---

### 8. Demo Script

Provide a runnable demo that shows:

- raw clinical note input
- protected prompt
- model-safe text
- final output
- metadata
- audit record

---

## MVP Out of Scope

The MVP should not include:

### 1. Full FHIR Integration

FHIR is important, but it should not be part of the first PHI MVP.

FHIR comes after the PHI boundary is proven.

---

### 2. FHIR Writes

No clinical write workflow in MVP.

---

### 3. Full HIPAA Compliance Claim

Do not claim the MVP is HIPAA compliant.

Use safer language:

    HIPAA-aware

or:

    designed to support HIPAA-aligned governance workflows

---

### 4. Expert Determination Engine

Expert Determination is out of scope for MVP.

---

### 5. Full Safe Harbor Certification

Safe Harbor mapping can be started, but do not claim complete Safe Harbor certification.

---

### 6. Reversible Pseudonymization

Do not build secure mapping storage in the first MVP.

Start with masking.

---

### 7. Multimodal PHI

No images, DICOM, voice, or waveform de-identification in MVP.

---

### 8. Clinical Safety Judgment

The MVP should not claim to validate clinical correctness.

It only governs PHI safety and runtime boundaries.

---

### 9. Autonomous Clinical Recommendations

No diagnosis, treatment, or medication recommendation workflows.

---

## MVP Components To Build

### 1. PHIScrubberGuard

Purpose:

    Healthcare-specific PHI masking guard.

Initial behavior:

- detect entities
- mask them
- return MODIFY
- attach MedMesh metadata

Potential implementation approach:

- wrap or subclass existing PIIScrubberGuard
- use Presidio as first backend
- add MedMesh metadata contract

---

### 2. PHIPromptBoundaryGuard

Purpose:

    Ensure no raw PHI remains before external LLM call.

Initial behavior:

- inspect prompt after PHIScrubberGuard
- if PHI-like entities remain, BLOCK
- attach policy metadata

---

### 3. PHILeakageOutputGuard

Purpose:

    Inspect model output before returning it.

Initial behavior:

- detect PHI in output
- block or mask based on config
- attach metadata

---

### 4. HealthcareAuditGuard

Purpose:

    Record governance decisions.

Initial behavior:

- write audit event to console or JSON file
- include no raw PHI
- include session_id and task_id when available
- include guard metadata

---

### 5. MedMesh External Strict Profile

Purpose:

    Define config that wires the MVP guards together.

Initial behavior:

- INPUT: structural, pattern, PHI
- PRE_LLM: PHI boundary
- OUTPUT: PHI leakage
- ASYNC: audit

---

## MVP Acceptance Criteria

The MVP is successful if it can demonstrate the following.

### Scenario 1: PHI Input Is Masked

Input:

    John Smith, MRN 123456, was admitted on March 3, 2024.

Expected protected text:

    [PERSON], [MEDICAL_RECORD_NUMBER], was admitted on [DATE].

Expected result:

- GuardVerdict.MODIFY
- phi_detected = true
- entity_count greater than 0
- handling_mode = mask

---

### Scenario 2: Clean Input Passes

Input:

    The patient reports mild headache and fatigue.

Expected result:

- GuardVerdict.PASS
- phi_detected = false

---

### Scenario 3: External Model Boundary Blocks Raw PHI

Input reaching PRE_LLM:

    John Smith was admitted yesterday.

Expected result:

- GuardVerdict.BLOCK
- reason_code = raw_phi_external_model_boundary
- audit_required = true

---

### Scenario 4: Output Leakage Is Caught

Model output:

    John Smith should follow up next week.

Expected result:

- output is blocked or masked
- PHI leakage metadata is attached
- audit event is generated

---

### Scenario 5: Audit Event Contains No Raw PHI

Audit event should include:

- audit_event_type
- phi_detected
- phi_types
- entity_count
- handling_mode
- policy_id
- session_id
- task_id

Audit event should not include:

- John Smith
- MRN 123456
- raw clinical note
- raw model output

---

## MVP Demo Input

Use synthetic test data only.

Example synthetic note:

    John Smith is a 67-year-old male admitted to Mercy General Hospital on March 3, 2024. His MRN is 123456. He presented with shortness of breath and elevated blood pressure. He was started on lisinopril and advised to follow up with Dr. Adams.

Expected masked note:

    [PERSON] is a 67-year-old male admitted to [ORGANIZATION] on [DATE]. His [MEDICAL_RECORD_NUMBER] is [ID]. He presented with shortness of breath and elevated blood pressure. He was started on lisinopril and advised to follow up with [PERSON].

Important:

The demo must clearly state that this is synthetic data.

---

## MVP Research Value

This MVP supports the research thesis:

    Runtime PHI governance for healthcare AI agents.

It demonstrates:

- PHI detection at runtime
- content transformation before model invocation
- model boundary enforcement
- output leakage inspection
- audit metadata generation
- profile-based governance

This can later contribute to a research paper on:

    Policy-aware runtime governance for healthcare AI agents.

---

## MVP Product Value

This MVP gives MedMesh a clear first product story:

    Developers can safely prototype healthcare AI agents without leaking PHI to external models.

This is easier to explain than a broad platform.

It also creates a foundation for:

- FHIR access governance
- patient-scoped authorization
- clinical review workflows
- observability dashboards
- enterprise deployment profiles

---

## Branching Guidance

Do not branch yet while writing architecture docs.

Branch when implementation begins.

Recommended branch for the first implementation:

    feature/medmesh-phi-gateway

Create it only after the following docs exist:

1. security-pipeline-analysis.md
2. core-runtime-contracts-analysis.md
3. guard-implementation-analysis.md
4. pii-and-phi-governance-analysis.md
5. governance-metadata-contracts.md
6. medmesh-pipeline-profiles.md
7. mvp-scope-and-demo-workflow.md

Once these are committed, implementation can begin on a feature branch.

---

## Next Recommended Step

After this document is committed, create one final planning document before branching:

    docs/roadmap/implementation-plan-phi-gateway.md

That document should break the MVP into concrete engineering tasks.

After that, create the implementation branch:

    git checkout -b feature/medmesh-phi-gateway

