# MedMesh Pipeline Profiles

## Objective

Define MedMesh runtime pipeline profiles for different healthcare AI deployment scenarios.

The existing AgentGuard runtime supports guard execution through intercept points and configurable guards.

MedMesh should build on that foundation by introducing healthcare-specific pipeline profiles.

A pipeline profile defines:

- which guards are enabled
- where they run
- whether they enforce, observe, or remain disabled
- how PHI is handled
- whether audit is required
- what type of healthcare workflow is supported
- what risk assumptions apply

---

## Why Pipeline Profiles Matter

Healthcare AI systems do not all have the same risk profile.

A research de-identification pipeline is different from a patient-facing assistant.

A clinician-facing internal assistant is different from a third-party external model workflow.

A FHIR read-only workflow is different from a FHIR write workflow.

Therefore, MedMesh should not have one universal default configuration.

Instead, MedMesh should define clear profiles.

---

## Core Principle

MedMesh should use profiles to make healthcare governance explicit.

The profile should answer:

- Is PHI allowed?
- Can PHI leave the trusted boundary?
- Can tools access FHIR?
- Can the agent write to clinical systems?
- Is human review required?
- Is this patient-facing or clinician-facing?
- Is output inspection required?
- Is audit required?
- Are semantic guards enabled?
- Are policies enforced or only observed?

---

## Existing Runtime Foundation

The verified AgentGuard runtime already supports:

- INPUT intercept point
- PRE_LLM intercept point
- TOOL_AUTH intercept point
- PRE_TOOL intercept point
- TOOL_OUTPUT intercept point
- OUTPUT intercept point
- ASYNC intercept point

It also supports guard modes:

- ENFORCE
- OBSERVE
- DISABLED

MedMesh profiles should use these existing mechanisms before introducing new runtime primitives.

---

## Proposed MedMesh Profiles

Initial MedMesh profiles:

1. medmesh-dev
2. medmesh-clinical-internal
3. medmesh-external-model-strict
4. medmesh-research-deidentification
5. medmesh-patient-facing
6. medmesh-fhir-readonly
7. medmesh-fhir-write-controlled

---

# 1. medmesh-dev

## Purpose

Used for local development, experimentation, testing, and guard tuning.

## Risk Assumption

No real PHI should be used.

If PHI is accidentally present, the system should detect it and record metadata, but the main goal is developer feedback.

## Intended Users

- developers
- researchers
- platform engineers
- local test environments

## Recommended Guard Mode

Mostly OBSERVE.

## PHI Handling

Detect only.

Do not modify content by default.

## Audit Requirement

Lightweight audit.

Console or local file audit is acceptable.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard
- PIIScrubberGuard in detect-only mode
- Future PHIScrubberGuard in detect-only mode

PRE_LLM:

- InjectionPatternGuard
- Future PromptPolicyGuard in observe mode

TOOL_AUTH:

- ToolScopeGuard in observe or enforce mode

PRE_TOOL:

- ToolArgumentSchemaGuard
- DangerousArgumentGuard

TOOL_OUTPUT:

- Future PHIScrubberGuard in detect-only mode

OUTPUT:

- Future PHILeakageOutputGuard in observe mode

ASYNC:

- Future HealthcareAuditGuard in local mode

## Expected Behavior

- Do not block aggressively.
- Surface what would have happened in stricter profiles.
- Help tune false positives and false negatives.
- Do not use real patient data.

---

# 2. medmesh-clinical-internal

## Purpose

Used for internal clinician-facing AI assistants inside a trusted healthcare environment.

Example workflows:

- clinical note summarization
- chart review assistance
- patient timeline generation
- prior authorization draft support
- clinician workflow assistant

## Risk Assumption

The system may process PHI inside a trusted clinical boundary.

The user is authenticated and authorized.

The model destination should be approved for PHI processing or protected through masking/pseudonymization.

## Intended Users

- clinicians
- care coordinators
- internal healthcare staff
- clinical operations teams

## Recommended Guard Mode

ENFORCE for deterministic safety controls.

OBSERVE for early semantic controls until validated.

## PHI Handling

Depends on model destination.

If model is approved for PHI:

- allow PHI inside trusted boundary
- audit all access
- inspect output for unauthorized leakage

If model is not approved for PHI:

- pseudonymize or mask before model call
- preserve mapping only inside trusted environment

## Audit Requirement

Mandatory.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard
- Future PHIScrubberGuard
- Future PatientContextGuard

PRE_LLM:

- Future PHIPromptBoundaryGuard
- Future ModelDestinationPolicyGuard
- Future PromptPolicyGuard

TOOL_AUTH:

- ToolScopeGuard
- Future FHIRScopeGuard
- Future ConsentContextGuard
- Future TenantBoundaryGuard

PRE_TOOL:

- ToolArgumentSchemaGuard
- DangerousArgumentGuard
- Future FHIROperationGuard

TOOL_OUTPUT:

- Future PHIScrubberGuard
- Future FHIRResourceSensitivityGuard

OUTPUT:

- Future PHILeakageOutputGuard
- Future ClinicalOutputSafetyGuard in observe mode first

ASYNC:

- Future HealthcareAuditGuard
- Future PHIAuditGuard
- Future FHIRAccessAuditGuard
- Future GovernanceTelemetryGuard

## Expected Behavior

- PHI may be processed if policy allows it.
- All clinical data access must be auditable.
- Tool access must be scoped.
- FHIR access must be patient-aware.
- High-risk actions should require review.
- Patient-facing output should not be generated from this profile unless explicitly configured.

---

# 3. medmesh-external-model-strict

## Purpose

Used when any part of the workflow sends prompts or context to a third-party or external model that is not approved to receive raw PHI.

## Risk Assumption

Raw PHI must not leave the trusted boundary.

## Intended Users

- healthcare AI developers using external LLM APIs
- organizations without BAA-covered model infrastructure
- research prototypes using hosted models
- controlled demos

## Recommended Guard Mode

ENFORCE.

## PHI Handling

Strict masking or anonymization before model call.

No raw PHI should reach the external model.

## Audit Requirement

Mandatory.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard
- Future PHIScrubberGuard in strict mode

PRE_LLM:

- Future PHIPromptBoundaryGuard
- Future ExternalModelPHIBlockGuard
- Future ModelDestinationPolicyGuard

TOOL_AUTH:

- ToolScopeGuard
- Future FHIRScopeGuard

PRE_TOOL:

- ToolArgumentSchemaGuard
- DangerousArgumentGuard
- Future BulkAccessGuard

TOOL_OUTPUT:

- Future PHIScrubberGuard in strict mode
- Future FHIRResourceSensitivityGuard

OUTPUT:

- Future PHILeakageOutputGuard in enforce mode
- Future ExternalOutputPolicyGuard

ASYNC:

- Future HealthcareAuditGuard
- Future PHIAuditGuard
- Future PolicyDecisionAuditGuard

## Expected Behavior

- Raw PHI is blocked or transformed before leaving the trusted boundary.
- Any detected PHI in final output is blocked or redacted.
- Tool outputs are inspected before entering model context.
- All PHI transformations are audited.
- Reversible mappings, if used, must remain inside trusted infrastructure.

---

# 4. medmesh-research-deidentification

## Purpose

Used for preparing healthcare data for research, analytics, evaluation, or model development.

## Risk Assumption

The output may be used outside direct care workflows.

Therefore, the system should prefer irreversible anonymization unless explicitly configured otherwise.

## Intended Users

- clinical researchers
- informatics teams
- data science teams
- AI evaluation teams
- research operations teams

## Recommended Guard Mode

ENFORCE.

## PHI Handling

Default to irreversible anonymization or Safe Harbor-style transformation.

## Audit Requirement

Mandatory.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- Future PHIScrubberGuard
- Future HIPAASafeHarborGuard
- Future DateGeneralizationGuard
- Future AgeGeneralizationGuard

PRE_LLM:

- Future ResearchDataPolicyGuard
- Future DeidentificationCompletenessGuard

TOOL_AUTH:

- ToolScopeGuard
- Future ResearchAccessScopeGuard

PRE_TOOL:

- ToolArgumentSchemaGuard
- DangerousArgumentGuard
- Future BulkAccessGuard

TOOL_OUTPUT:

- Future PHIScrubberGuard
- Future HIPAASafeHarborGuard
- Future DeidentificationCompletenessGuard

OUTPUT:

- Future PHILeakageOutputGuard
- Future DeidentifiedOutputValidator

ASYNC:

- Future HealthcareAuditGuard
- Future DeidentificationAuditGuard
- Future ResidualRiskTelemetryGuard

## Expected Behavior

- PHI is transformed or removed.
- Safe Harbor categories are mapped in metadata.
- Re-identification risk should be tracked.
- Raw PHI should not be retained in outputs.
- Utility-preserving strategies may be added later.
- Expert Determination workflows are future phase, not MVP.

---

# 5. medmesh-patient-facing

## Purpose

Used for AI systems that generate output directly for patients or caregivers.

Example workflows:

- patient education
- visit summary explanation
- care navigation
- medication instruction clarification
- appointment preparation

## Risk Assumption

Patient-facing output carries elevated risk.

The system must avoid unsafe medical advice, confusing recommendations, unauthorized PHI exposure, or clinician-like certainty where not appropriate.

## Intended Users

- patients
- caregivers
- patient support teams
- consumer-facing healthcare apps

## Recommended Guard Mode

ENFORCE for PHI and policy checks.

OBSERVE or ENFORCE for clinical safety depending on maturity.

## PHI Handling

Patient-specific PHI may be allowed only if:

- the patient is authenticated
- patient context matches
- consent and authorization are valid
- output is intended for that patient

Otherwise, PHI must be blocked or redacted.

## Audit Requirement

Mandatory.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard
- Future PatientContextGuard
- Future PHIScrubberGuard

PRE_LLM:

- Future PatientFacingPromptPolicyGuard
- Future PHIPromptBoundaryGuard

TOOL_AUTH:

- ToolScopeGuard
- Future PatientScopedFHIRGuard
- Future ConsentContextGuard

PRE_TOOL:

- ToolArgumentSchemaGuard
- DangerousArgumentGuard
- Future FHIROperationGuard

TOOL_OUTPUT:

- Future PHIScrubberGuard
- Future PatientScopeMatchGuard

OUTPUT:

- Future PHILeakageOutputGuard
- Future PatientFacingSafetyGuard
- Future MedicalDisclaimerGuard
- Future HumanReviewGuard for high-risk outputs

ASYNC:

- Future HealthcareAuditGuard
- Future PatientOutputAuditGuard
- Future ClinicalSafetyTelemetryGuard

## Expected Behavior

- Patient-facing output must be safer than clinician-facing output.
- High-risk recommendations should be blocked or reviewed.
- Medical disclaimers may be required.
- Output should be understandable and not overstate certainty.
- Any patient-specific PHI must match the authenticated patient context.

---

# 6. medmesh-fhir-readonly

## Purpose

Used when agents can read from FHIR systems but cannot write.

Example workflows:

- chart summarization
- lab trend explanation
- medication list review
- encounter timeline generation
- prior authorization evidence gathering

## Risk Assumption

Read-only access is safer than write access, but still sensitive because it can expose PHI.

## Intended Users

- clinician assistants
- internal agents
- research assistants
- administrative workflow agents

## Recommended Guard Mode

ENFORCE.

## PHI Handling

Inspect FHIR outputs before they enter model context.

Apply policy based on model destination and user authorization.

## Audit Requirement

Mandatory.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard

PRE_LLM:

- Future PHIPromptBoundaryGuard
- Future ModelDestinationPolicyGuard

TOOL_AUTH:

- ToolScopeGuard
- Future FHIRScopeGuard
- Future SMARTScopeGuard
- Future PatientContextGuard

PRE_TOOL:

- ToolArgumentSchemaGuard
- Future FHIRReadOperationGuard
- Future PatientScopeMatchGuard

TOOL_OUTPUT:

- Future FHIRResourceSensitivityGuard
- Future PHIScrubberGuard
- Future MinimumNecessaryGuard

OUTPUT:

- Future PHILeakageOutputGuard
- Future ClinicalOutputSafetyGuard in observe mode first

ASYNC:

- Future FHIRAccessAuditGuard
- Future HealthcareAuditGuard
- Future PolicyDecisionAuditGuard

## Expected Behavior

- Read access must be scoped.
- Patient context must match authorization.
- FHIR resource type and operation must be recorded.
- Bulk access should be restricted.
- Tool outputs should be classified before model use.

---

# 7. medmesh-fhir-write-controlled

## Purpose

Used when agents can propose or perform writes to FHIR systems under strict governance.

Example workflows:

- draft care plan updates
- propose appointment updates
- draft patient message
- suggest medication reconciliation changes
- update non-critical administrative fields

## Risk Assumption

FHIR writes are high-risk.

Autonomous clinical writes should not be allowed in early MedMesh versions.

## Intended Users

- clinicians
- supervised clinical agents
- administrative healthcare workflows
- internal clinical operations teams

## Recommended Guard Mode

ENFORCE.

## PHI Handling

PHI may be present in the trusted boundary but must be audited and scoped.

## Audit Requirement

Mandatory and high-detail.

## Recommended Guards

INPUT:

- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- InjectionPatternGuard
- JailbreakPhraseGuard
- Future PHIScrubberGuard
- Future PatientContextGuard

PRE_LLM:

- Future PHIPromptBoundaryGuard
- Future ClinicalWorkflowPolicyGuard

TOOL_AUTH:

- ToolScopeGuard
- Future FHIRScopeGuard
- Future SMARTScopeGuard
- Future ClinicianRoleGuard
- Future ConsentContextGuard

PRE_TOOL:

- ToolArgumentSchemaGuard
- DangerousArgumentGuard
- Future FHIRWriteOperationGuard
- Future HumanReviewGuard
- Future ClinicalWriteApprovalGuard

TOOL_OUTPUT:

- Future FHIRWriteResultAuditGuard
- Future PHIScrubberGuard

OUTPUT:

- Future ClinicalOutputSafetyGuard
- Future PatientFacingSafetyGuard if patient-facing
- Future PHILeakageOutputGuard

ASYNC:

- Future FHIRWriteAuditGuard
- Future HealthcareAuditGuard
- Future GovernanceTelemetryGuard

## Expected Behavior

- Writes require strict authorization.
- High-risk writes require human review.
- Autonomous clinical writes should be blocked by default.
- All write attempts must be audited.
- The system should distinguish draft generation from actual FHIR write execution.

---

## Profile Comparison Matrix

| Profile | PHI Allowed | External Model Safe | FHIR Read | FHIR Write | Human Review | Audit |
|---|---|---|---|---|---|---|
| medmesh-dev | No real PHI | Not guaranteed | Optional mock | No | Optional | Lightweight |
| medmesh-clinical-internal | Yes, if authorized | Depends on model policy | Yes | Limited | For high-risk actions | Mandatory |
| medmesh-external-model-strict | No raw PHI | Yes | Yes, transformed | No by default | For exceptions | Mandatory |
| medmesh-research-deidentification | Input may contain PHI | Output should be de-identified | Optional | No | Optional expert review | Mandatory |
| medmesh-patient-facing | Only patient-scoped | Depends on model policy | Yes, scoped | No by default | For high-risk output | Mandatory |
| medmesh-fhir-readonly | Yes, if authorized | Depends on model policy | Yes | No | Usually no | Mandatory |
| medmesh-fhir-write-controlled | Yes, if authorized | Depends on model policy | Yes | Controlled | Yes | Mandatory |

---

## Default MVP Profile

The recommended first MVP profile is:

    medmesh-external-model-strict

Reason:

This profile is easiest to explain and safest to demonstrate.

It shows that MedMesh can:

- detect PHI
- protect prompts before external model calls
- inspect tool outputs
- enforce tool scope
- audit governance decisions
- block unsafe leakage

It also aligns strongly with the product narrative:

    Governed healthcare AI agents that do not leak PHI.

---

## First Demo Workflow

Recommended first demo:

    Secure Clinical Note Summarization With External Model Protection

Flow:

1. User submits clinical note.
2. INPUT guard validates structure.
3. PHI guard detects and masks PHI.
4. PRE_LLM guard verifies no raw PHI leaves trusted boundary.
5. LLM summarizes de-identified note.
6. OUTPUT guard checks for PHI leakage.
7. ASYNC audit guard records transformation and policy decisions.

This demo is narrow, safe, understandable, and research-relevant.

---

## Configuration Direction

MedMesh should eventually support profile-based configuration.

Example conceptual profile configuration:

    profile: medmesh-external-model-strict
    phi:
      handling_mode: mask
      safe_harbor_mapping: true
      output_inspection: true
    model:
      destination: external
      allow_raw_phi: false
    audit:
      enabled: true
      sink: otel
    fhir:
      enabled: false
    clinical_safety:
      mode: observe

This is conceptual and should not be treated as current implementation.

---

## Implementation Strategy

### Phase 1

Do not build all profiles.

Document all profiles first.

### Phase 2

Implement only:

    medmesh-external-model-strict

### Phase 3

Add:

    medmesh-fhir-readonly

### Phase 4

Add:

    medmesh-clinical-internal

### Phase 5

Add:

    medmesh-research-deidentification

### Phase 6

Add:

    medmesh-patient-facing
    medmesh-fhir-write-controlled

---

## Relationship To HealthClaw Analysis

The HealthClaw analysis showed that FHIR/MCP proxy guardrails are an existing adjacent pattern.

MedMesh should therefore avoid being only a FHIR proxy.

Pipeline profiles help differentiate MedMesh because they govern the full agent runtime lifecycle, not only FHIR access.

MedMesh profiles cover:

- prompt governance
- model boundary governance
- PHI transformation
- tool authorization
- FHIR access
- final output inspection
- audit and telemetry
- deployment risk posture

---

## Principal Engineer Recommendation

MedMesh should introduce profile-based governance early.

This gives the platform:

- clarity
- safety
- configurability
- research structure
- enterprise relevance
- better demos
- easier documentation

The first implemented profile should be:

    medmesh-external-model-strict

because it provides the clearest and safest MVP story.

---

## Next Recommended Document

Create:

    docs/architecture/mvp-scope-and-demo-workflow.md

This document should define:

- MVP scope
- first demo workflow
- what to build
- what not to build
- acceptance criteria
- architecture sequence
- research value

