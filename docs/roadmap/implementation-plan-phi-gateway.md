# Implementation Plan: MedMesh PHI Gateway

## Objective

Define the concrete engineering implementation plan for the first MedMesh MVP:

    PHI-safe clinical note summarization with external model protection.

This plan converts the architecture docs into actionable implementation tasks.

---

## MVP Profile

The first implemented profile is:

    medmesh-external-model-strict

---

## MVP Goal

Prove that MedMesh can protect healthcare AI workflows by:

- detecting PHI
- masking PHI before external model use
- preventing raw PHI from crossing model boundaries
- inspecting model outputs for PHI leakage
- producing governance metadata
- emitting audit events without raw PHI

---

## Current Foundation

The existing AgentGuard runtime already provides:

- AgentContext
- GuardVerdict
- GuardResult
- GuardMode
- InterceptPoint
- SecurityGuard
- SecurityPipeline
- PipelineResult
- structural guards
- pattern guards
- budget guards
- scope guards
- PIIScrubberGuard using Presidio

---

## Implementation Strategy

Do not rewrite the runtime core.

Implement MedMesh as healthcare-specific extensions on top of the current guard pipeline.

The first implementation should add:

1. PHIScrubberGuard
2. PHIPromptBoundaryGuard
3. PHILeakageOutputGuard
4. HealthcareAuditGuard
5. medmesh-external-model-strict profile
6. synthetic clinical note summarization demo
7. basic tests

---

# Phase 1: Project Structure

## Goal

Create MedMesh-specific directories without renaming the existing agentguard package yet.

## Proposed Structure

    src/agentguard/medmesh/
    ├── __init__.py
    ├── guards/
    │   ├── __init__.py
    │   ├── phi.py
    │   ├── boundary.py
    │   └── audit.py
    ├── profiles/
    │   ├── __init__.py
    │   └── external_model_strict.py
    └── metadata/
        ├── __init__.py
        └── keys.py

## Tasks

- Create medmesh extension package.
- Create guard modules.
- Create profile module.
- Create metadata key constants.
- Keep existing package stable.

## Commands

    mkdir -p src/agentguard/medmesh/{guards,profiles,metadata}
    touch src/agentguard/medmesh/__init__.py
    touch src/agentguard/medmesh/guards/__init__.py
    touch src/agentguard/medmesh/guards/phi.py
    touch src/agentguard/medmesh/guards/boundary.py
    touch src/agentguard/medmesh/guards/audit.py
    touch src/agentguard/medmesh/profiles/__init__.py
    touch src/agentguard/medmesh/profiles/external_model_strict.py
    touch src/agentguard/medmesh/metadata/__init__.py
    touch src/agentguard/medmesh/metadata/keys.py

---

# Phase 2: Metadata Constants

## Goal

Prevent ad hoc metadata key usage.

## File

    src/agentguard/medmesh/metadata/keys.py

## Initial Constants

Create constants for:

- phi_detected
- phi_types
- pii_types
- entity_count
- entity_summary
- confidence_threshold
- detector_backend
- handling_mode
- anonymization_operator
- safe_harbor_applied
- safe_harbor_categories
- policy_id
- policy_version
- policy_decision
- audit_required
- risk_level
- reason_code
- human_readable_reason
- audit_event_id
- audit_event_type
- session_id
- task_id

## Acceptance Criteria

- All new MedMesh guards use metadata constants.
- No raw PHI metadata keys are introduced.
- Metadata keys align with governance-metadata-contracts.md.

---

# Phase 3: PHIScrubberGuard

## Goal

Create a MedMesh-specific PHI guard that builds on the existing PII foundation.

## File

    src/agentguard/medmesh/guards/phi.py

## Guard Name

    PHIScrubberGuard

## Intercept Points

Initial default intercept points:

- INPUT
- TOOL_OUTPUT

Later add:

- OUTPUT

## Behavior

If input is empty:

- return PASS

If no PHI or PII detected:

- return PASS with metadata:
  - phi_detected = false
  - entity_count = 0

If PHI or PII detected:

- transform content according to handling_mode
- return MODIFY
- include metadata:
  - phi_detected = true
  - phi_types
  - entity_count
  - entity_summary
  - handling_mode
  - detector_backend
  - anonymization_operator
  - policy_id
  - audit_required
  - risk_level

## Initial Handling Mode

Only implement:

    mask

Do not implement:

- pseudonymize
- date shifting
- reversible mapping
- semantic replacement

## Backend

Use Presidio first if available.

If Presidio is unavailable:

- fail clearly during setup
- do not silently disable PHI protection

## Acceptance Criteria

- Synthetic note with names and dates returns MODIFY.
- Clean input returns PASS.
- Metadata does not contain raw PHI.
- Guard fail-closes on internal error.
- Guard can be configured with threshold and handling_mode.

---

# Phase 4: PHIPromptBoundaryGuard

## Goal

Prevent raw PHI from reaching an external model boundary.

## File

    src/agentguard/medmesh/guards/boundary.py

## Guard Name

    PHIPromptBoundaryGuard

## Intercept Point

- PRE_LLM

## Behavior

Inspect prompt before model invocation.

If PHI-like content is detected:

- return BLOCK
- include metadata:
  - phi_detected = true
  - policy_id = model.external.no_raw_phi.v1
  - policy_decision = deny
  - reason_code = raw_phi_external_model_boundary
  - audit_required = true
  - risk_level = high

If no PHI is detected:

- return PASS
- include metadata:
  - phi_detected = false
  - policy_decision = allow

## Acceptance Criteria

- Raw name/date prompt is blocked.
- Masked prompt passes.
- Metadata explains the policy decision.
- No raw PHI is stored in metadata.

---

# Phase 5: PHILeakageOutputGuard

## Goal

Detect PHI leakage in final model output.

## File

    src/agentguard/medmesh/guards/boundary.py

## Guard Name

    PHILeakageOutputGuard

## Intercept Point

- OUTPUT

## Behavior

Inspect model output before returning it.

If PHI is detected:

Initial MVP behavior:

- return BLOCK

Alternative later behavior:

- return MODIFY with redacted output

For MVP, blocking is safer.

Metadata:

- phi_detected
- phi_types
- entity_count
- policy_id = output.no_phi_leakage.v1
- policy_decision
- reason_code
- audit_required
- risk_level

## Acceptance Criteria

- Output containing synthetic person name is blocked.
- Clean summary passes.
- Metadata is attached.
- No raw PHI appears in metadata.

---

# Phase 6: HealthcareAuditGuard

## Goal

Emit audit events for MedMesh governance decisions.

## File

    src/agentguard/medmesh/guards/audit.py

## Guard Name

    HealthcareAuditGuard

## Intercept Point

- ASYNC

## Initial Audit Sink

Use structured JSON lines.

Options:

- stdout
- local file

For MVP, stdout is acceptable.

## Audit Event Must Include

- audit_event_id
- audit_event_type
- session_id
- task_id
- guard_name
- verdict
- policy_id
- policy_decision
- phi_detected
- phi_types
- entity_count
- handling_mode
- risk_level
- timestamp

## Audit Event Must Not Include

- raw clinical note
- raw prompt
- raw model output
- raw PHI values
- medical record number
- patient name
- phone number
- email address

## Acceptance Criteria

- Audit event is emitted.
- Audit event is JSON serializable.
- Audit event excludes raw PHI.
- Audit event includes session_id and task_id when available.

---

# Phase 7: External Model Strict Profile

## Goal

Create a profile factory that wires the MVP guards into a SecurityPipeline.

## File

    src/agentguard/medmesh/profiles/external_model_strict.py

## Profile Name

    medmesh-external-model-strict

## Required Guards

INPUT:

- existing structural guards
- existing pattern guards
- PHIScrubberGuard

PRE_LLM:

- PHIPromptBoundaryGuard

OUTPUT:

- PHILeakageOutputGuard

ASYNC:

- HealthcareAuditGuard

## Optional Existing Guards

May include:

- budget guards
- scope guards

## Acceptance Criteria

- Profile returns a SecurityPipeline.
- Profile includes PHI guard.
- Profile includes PRE_LLM boundary guard.
- Profile includes OUTPUT leakage guard.
- Profile includes ASYNC audit guard.

---

# Phase 8: Demo

## Goal

Create a runnable synthetic demo.

## File

    examples/medmesh_phi_gateway_demo.py

## Demo Input

Use synthetic data only.

Example:

    John Smith is a 67-year-old male admitted to Mercy General Hospital on March 3, 2024. His MRN is 123456. He presented with shortness of breath and elevated blood pressure. He was started on lisinopril and advised to follow up with Dr. Adams.

## Demo Should Print

- original synthetic note
- protected note after INPUT pipeline
- PRE_LLM boundary result
- mock model summary
- OUTPUT pipeline result
- governance metadata
- audit event

## Important

The demo should not call a real external LLM initially.

Use a mock summarizer first.

Reason:

- avoids dependency complexity
- avoids accidental PHI exposure
- keeps MVP deterministic

## Acceptance Criteria

- Demo runs locally.
- Demo does not require external API keys.
- Demo shows PHI masking.
- Demo shows metadata.
- Demo shows audit event.
- Demo uses synthetic data only.

---

# Phase 9: Tests

## Goal

Create tests for the MVP guards and profile.

## Proposed Test Files

    tests/medmesh/test_phi_guard.py
    tests/medmesh/test_boundary_guards.py
    tests/medmesh/test_audit_guard.py
    tests/medmesh/test_external_model_strict_profile.py

## Test Cases

### PHIScrubberGuard

- clean input passes
- PHI input modifies
- metadata contains phi_detected
- metadata excludes raw PHI
- empty input passes

### PHIPromptBoundaryGuard

- masked prompt passes
- raw PHI prompt blocks
- metadata contains policy decision

### PHILeakageOutputGuard

- clean output passes
- PHI output blocks
- metadata contains leakage reason

### HealthcareAuditGuard

- emits JSON
- excludes raw PHI
- includes session_id and task_id

### Profile

- builds SecurityPipeline
- includes expected guard types
- can run INPUT, PRE_LLM, OUTPUT, and ASYNC stages

---

# Phase 10: Documentation Updates

## Update README

Add:

- MedMesh section
- MVP summary
- demo command
- safety disclaimer

## Add Example Documentation

Create:

    docs/examples/phi-safe-clinical-summarization.md

Include:

- what the demo does
- what it does not do
- how to run it
- expected output
- safety assumptions

---

# Safety Language

Do not claim:

- HIPAA compliant
- clinically validated
- safe for production
- certified de-identification
- diagnostic AI

Use safer language:

- HIPAA-aware
- PHI-aware
- designed to support governance workflows
- research prototype
- synthetic data demo
- not for clinical use

---

# Branching Plan

After this implementation plan is committed, create a branch:

    git checkout -b feature/medmesh-phi-gateway

All implementation work for the first MVP should happen on that branch.

Do not implement directly on the documentation baseline branch.

---

# Definition Of Done

The PHI Gateway MVP is complete when:

1. MedMesh extension package exists.
2. Metadata constants exist.
3. PHIScrubberGuard works with synthetic text.
4. PRE_LLM boundary guard blocks raw PHI.
5. OUTPUT leakage guard catches PHI leakage.
6. HealthcareAuditGuard emits safe audit JSON.
7. External model strict profile builds a pipeline.
8. Demo runs with mock summarizer.
9. Tests pass.
10. README and demo docs are updated.

---

# Next Step After MVP

After PHI Gateway MVP:

    implement FHIR read-only governance

Potential future branch:

    feature/medmesh-fhir-readonly-governance

