# PII and PHI Governance Analysis

## Objective

Analyze the existing PII capability in AgentGuard and define how it should evolve into MedMesh PHI governance for healthcare AI agents.

This document focuses on:

- current PII detection behavior
- current anonymization behavior
- current limitations
- healthcare-specific PHI requirements
- HIPAA Safe Harbor implications
- future MedMesh PHI governance design
- MVP implementation direction

---

## Source Files Reviewed

The verified current implementation is located in:

    src/agentguard/security/guards/classifiers.py

The relevant class is:

    PIIScrubberGuard

The project dependency configuration is located in:

    pyproject.toml

The relevant optional dependency group is:

    presidio

---

## Current Implementation Summary

The current codebase includes a PII guard called:

    PIIScrubberGuard

It is implemented as a category 5 guard.

Its default intercept points are:

    INPUT
    TOOL_OUTPUT

Its default timeout is:

    200 ms

This means the guard is intended to inspect:

- incoming user input
- data returned from tools

This is a strong foundation for MedMesh because PHI governance must happen both before model invocation and after tool retrieval.

---

## Current PIIScrubberGuard Behavior

### Setup Behavior

PIIScrubberGuard attempts to import:

- presidio_analyzer.AnalyzerEngine
- presidio_anonymizer.AnonymizerEngine

If those packages are not installed, it raises AgentGuardConfigurationError with an instruction to install:

    agentguard[presidio]

The optional dependencies in pyproject.toml include:

- presidio-analyzer
- presidio-anonymizer
- spacy

This confirms that Presidio is the current PII detection and anonymization backend.

---

## Current Configuration

PIIScrubberGuard currently reads:

    sensitivity_threshold

Default value:

    0.7

It also reads:

    language

Default value:

    en

---

## Current Check Flow

The current check behavior is:

1. If the input is empty or whitespace, return PASS.
2. Analyze the input using Presidio AnalyzerEngine.
3. Use the configured language and score threshold.
4. If no PII entities are found, return PASS.
5. If PII entities are found, anonymize the input using Presidio AnonymizerEngine.
6. Return MODIFY with the anonymized text.
7. Attach metadata containing:
   - pii_types
   - entity_count

---

## Current GuardResult Behavior

When PII is detected, PIIScrubberGuard returns:

    GuardVerdict.MODIFY

This is important.

It does not block the request by default.

Instead, it transforms the request and allows the pipeline to continue with anonymized text.

This behavior is well aligned with the MedMesh PHI Gateway concept.

---

## Current Metadata

The current metadata includes:

- pii_types
- entity_count

Example meaning:

    pii_types = ["PERSON", "PHONE_NUMBER"]
    entity_count = 2

This is useful but insufficient for healthcare auditability.

---

## Current Strengths

### 1. Existing PII Guard Exists

The current codebase already includes a PII guard.

This means MedMesh does not need to start from zero.

---

### 2. Uses Presidio

Presidio is a practical starting point for PII detection and anonymization.

It supports analyzer and anonymizer workflows, which map naturally to detection and transformation pipelines.

---

### 3. MODIFY Verdict Is Already Supported

The current runtime can already propagate modified input.

This is essential for PHI redaction, masking, or anonymization.

---

### 4. Intercepts INPUT and TOOL_OUTPUT

This is strategically important.

For healthcare AI systems, sensitive data may enter from:

- user prompts
- clinical notes
- tool outputs
- FHIR responses
- retrieved documents
- patient messages

Inspecting both input and tool output is a strong starting point.

---

### 5. Metadata Already Exists

The metadata field provides an initial audit extension point.

MedMesh can use this to record PHI governance details before introducing a richer audit model.

---

## Current Limitations

### 1. General PII, Not Healthcare PHI

The current guard is general-purpose PII handling.

It is not yet healthcare-grade PHI governance.

It does not explicitly model HIPAA Safe Harbor identifiers.

---

### 2. No HIPAA Safe Harbor Policy

The current implementation does not define the 18 HIPAA Safe Harbor identifier categories.

MedMesh will need explicit support for these categories.

---

### 3. No Clinical Context Awareness

The current guard does not understand clinical context.

Example challenge:

    "Smith fracture"

should not be treated the same way as:

    "Dr. Smith"

Clinical context matters.

---

### 4. No Date Generalization Strategy

Healthcare de-identification often requires special handling for dates.

Current implementation does not define:

- removing day/month while preserving year
- shifting dates consistently
- generalizing dates to periods
- preserving chronology while reducing identifiability

---

### 5. No Age Over 89 Handling

HIPAA Safe Harbor requires special treatment of ages over 89.

The current implementation does not explicitly handle this.

---

### 6. No Medical Identifier Strategy

The current implementation does not explicitly distinguish healthcare-specific identifiers such as:

- medical record numbers
- health plan beneficiary numbers
- account numbers
- certificate or license numbers
- device identifiers
- biometric identifiers

---

### 7. No Reversible Tokenization Strategy

The current Presidio anonymization approach transforms text, but MedMesh has not yet defined whether it needs:

- irreversible anonymization
- reversible tokenization
- consistent pseudonymization
- semantically preserving replacement

This is a major architectural decision.

---

### 8. No Audit Event Model

The current guard metadata is minimal.

It does not yet record:

- entity spans
- entity confidence scores
- anonymization operator used
- original value hash
- replacement value type
- policy ID
- PHI handling mode
- patient context
- workflow context

---

### 9. Not Enabled By Default

Based on the verified default SecurityPipeline.from_config behavior, PIIScrubberGuard exists but is not part of the default instantiated guard set.

This is important.

MedMesh must make PHI governance first-class, not optional or hidden.

---

## Healthcare PHI Requirements

MedMesh PHI governance must eventually support HIPAA Safe Harbor-style identifiers.

Key identifier categories include:

- names
- geographic subdivisions smaller than a state
- dates directly related to an individual, except year
- telephone numbers
- fax numbers
- email addresses
- social security numbers
- medical record numbers
- health plan beneficiary numbers
- account numbers
- certificate and license numbers
- vehicle identifiers and serial numbers
- device identifiers and serial numbers
- URLs
- IP addresses
- biometric identifiers
- full-face photographic images and comparable images
- any other unique identifying number, characteristic, or code

---

## Safe Harbor vs Expert Determination

MedMesh should treat de-identification as a governance mode, not a single static function.

Two major modes should be considered:

### Safe Harbor Mode

A rules-based mode focused on removing or transforming the HIPAA Safe Harbor identifier categories.

Benefits:

- clearer implementation
- easier documentation
- easier auditability

Limitations:

- may reduce data utility
- may be too rigid for advanced AI workflows

### Expert Determination Mode

A more flexible, context-sensitive mode focused on reducing re-identification risk to a very low level.

Benefits:

- preserves more utility
- better for advanced healthcare AI workflows
- more adaptive to context

Limitations:

- requires expert-defined methodology
- requires stronger documentation
- requires validation and periodic re-evaluation

---

## Full Anonymization vs Pseudonymization

MedMesh must distinguish between:

### Full Anonymization

Goal:

    Irreversibly remove or transform identifiers so re-identification is not reasonably possible.

Best for:

- research datasets
- external sharing
- model training datasets
- public or third-party data use

### Pseudonymization

Goal:

    Replace identifiers with tokens while preserving a re-identification path through a secure mapping.

Best for:

- internal workflows
- patient-specific continuity
- temporary prompt protection
- reversible clinical operations

Important:

Pseudonymized data may still be considered personal data under privacy regimes such as GDPR because re-identification remains possible through supplementary information.

---

## MedMesh PHI Governance Modes

MedMesh should eventually support multiple PHI handling modes.

### 1. Detect Only

Purpose:

    Identify PHI but do not modify content.

Use cases:

- observe mode
- evaluation
- false-positive analysis
- compliance review

Expected verdict:

    PASS

Expected metadata:

- phi_detected
- phi_types
- entity_count
- confidence_scores

---

### 2. Mask

Purpose:

    Replace detected PHI with placeholders.

Example:

    John Smith visited on March 3, 2024.

Becomes:

    [PERSON] visited on [DATE].

Expected verdict:

    MODIFY

---

### 3. Redact

Purpose:

    Remove sensitive values entirely.

Example:

    John Smith visited on March 3, 2024.

Becomes:

    [REDACTED] visited on [REDACTED].

Expected verdict:

    MODIFY

---

### 4. Pseudonymize

Purpose:

    Replace identifiers with stable tokens.

Example:

    John Smith

Becomes:

    PATIENT_001

Expected verdict:

    MODIFY

Important:

This requires secure mapping storage if reversibility is needed.

---

### 5. Semantically Preserving Replacement

Purpose:

    Replace identifiers with realistic but fake alternatives while preserving narrative coherence.

Example:

    John Smith called his wife Mary.

Becomes:

    Michael Turner called his wife Sarah.

Expected verdict:

    MODIFY

This is useful for LLM summarization and synthetic clinical text workflows.

---

### 6. Block

Purpose:

    Stop execution if high-risk PHI is detected in an unsafe context.

Examples:

- PHI sent to non-approved external model
- patient record exposed to unauthorized agent
- bulk PHI extraction attempt
- PHI detected in public output

Expected verdict:

    BLOCK

---

## Recommended MedMesh PHI Metadata Contract

For the MVP, use GuardResult.metadata with documented keys.

Suggested metadata keys:

- phi_detected
- phi_types
- pii_types
- entity_count
- entity_summary
- confidence_threshold
- handling_mode
- anonymization_operator
- safe_harbor_categories
- residual_risk_level
- policy_id
- policy_version
- reversible
- mapping_id
- patient_context_present
- audit_required

Example:

    {
      "phi_detected": true,
      "phi_types": ["PERSON", "DATE", "MEDICAL_RECORD_NUMBER"],
      "entity_count": 4,
      "handling_mode": "mask",
      "safe_harbor_categories": ["names", "dates", "medical_record_numbers"],
      "policy_id": "phi.safe_harbor.v1",
      "reversible": false,
      "audit_required": true
    }

---

## Proposed MedMesh PHI Components

### 1. PHIScrubberGuard

Purpose:

    Healthcare-specific successor or wrapper around PIIScrubberGuard.

Responsibilities:

- detect PHI
- classify PHI by healthcare category
- transform content according to configured mode
- attach PHI metadata
- preserve runtime compatibility with GuardResult.MODIFY

---

### 2. HIPAASafeHarborGuard

Purpose:

    Enforce HIPAA Safe Harbor-style handling.

Responsibilities:

- map detected entities to Safe Harbor categories
- enforce required suppression/generalization
- block unsupported high-risk identifiers
- document compliance behavior

---

### 3. ClinicalContextPHIGuard

Purpose:

    Reduce false positives and false negatives in clinical text.

Responsibilities:

- distinguish clinical terms from names
- detect implicit identifiers
- handle provider names
- handle facility references
- preserve clinically important meaning where possible

---

### 4. DateGeneralizationGuard

Purpose:

    Apply healthcare-specific date transformation.

Responsibilities:

- remove day/month where required
- preserve year when allowed
- shift dates consistently
- preserve clinical chronology
- handle admission, discharge, birth, death, and service dates

---

### 5. AgeGeneralizationGuard

Purpose:

    Handle age-related identifiers.

Responsibilities:

- detect ages over 89
- generalize to 90+
- preserve medically useful age bands where appropriate

---

### 6. PHIAuditGuard

Purpose:

    Record PHI governance decisions.

Responsibilities:

- record detected categories
- record transformation mode
- record policy ID
- record reversible or irreversible status
- record audit metadata without storing raw PHI

---

## MVP Recommendation

The first MedMesh PHI MVP should be narrow and deterministic.

Build:

1. PHIScrubberGuard
2. HIPAASafeHarbor metadata mapping
3. configurable handling mode
4. GuardResult.metadata enrichment
5. INPUT and TOOL_OUTPUT inspection
6. OUTPUT inspection for leakage prevention
7. basic audit record generation

Do not start with:

- full expert determination engine
- multimodal PHI handling
- DICOM de-identification
- full synthetic data generation
- complex semantic replacement

Those are later phases.

---

## Proposed MVP Behavior

### INPUT

Inspect user input before it reaches the LLM.

If PHI is detected:

- mask or pseudonymize based on config
- return MODIFY
- attach PHI metadata

### TOOL_OUTPUT

Inspect data returned from tools before it enters model context.

If PHI is detected:

- enforce policy based on tool, user, and context
- mask or block as needed
- attach audit metadata

### OUTPUT

Inspect final model output before it is shown to the user.

If unauthorized PHI appears:

- block or redact
- attach leakage metadata
- trigger audit event

---

## Example MedMesh PHI Policy Modes

### Development Mode

Behavior:

- detect only
- do not modify
- log metadata

Use case:

    local development and evaluation

---

### Internal Clinical Mode

Behavior:

- pseudonymize when sending to LLM
- allow re-identification only inside trusted clinical workflow
- audit all transformations

Use case:

    clinician-facing assistant inside approved environment

---

### Research Mode

Behavior:

- irreversible anonymization
- Safe Harbor-style transformation
- no mapping table

Use case:

    research dataset preparation and external sharing

---

### External Model Mode

Behavior:

- strict masking
- block high-risk identifiers
- no raw PHI leaves trusted boundary

Use case:

    third-party hosted model APIs

---

## Architecture Implications

### 1. AgentContext May Need Healthcare Metadata

Current AgentContext does not include patient or workflow context.

PHI governance will eventually need:

- patient_id
- encounter_id
- clinician_id
- organization_id
- consent_context
- workflow_id
- model_destination
- data_use_purpose

Recommendation:

Do not modify AgentContext immediately.

First use GuardResult.metadata and configuration.

Later introduce HealthcareContext or MedMeshExecutionContext.

---

### 2. GuardResult Metadata Must Be Standardized

The current metadata field is flexible but untyped.

For PHI governance, inconsistent metadata will create audit problems.

Recommendation:

Create a metadata convention document before implementation.

Suggested next document:

    docs/architecture/governance-metadata-contracts.md

---

### 3. Pipeline Profiles Are Needed

The default AgentGuard pipeline does not include PIIScrubberGuard.

MedMesh should define healthcare pipeline profiles.

Example profiles:

- medmesh-dev
- medmesh-clinical-internal
- medmesh-research
- medmesh-external-model-strict

Each profile should specify:

- enabled guards
- intercept points
- handling modes
- thresholds
- audit requirements

---

## Research Relevance

This PHI governance layer is central to the MedMesh research thesis.

Potential research framing:

    Runtime PHI governance for agentic healthcare AI systems.

The research contribution is not merely PHI detection.

The stronger contribution is:

    policy-aware, runtime PHI governance across AI agent execution stages.

This includes:

- input inspection
- tool-output inspection
- final-output inspection
- transformation provenance
- auditability
- healthcare-specific governance modes
- privacy-utility trade-off management

---

## Key Risks

### 1. False Negatives

Risk:

    PHI is missed and sent to an unsafe destination.

Mitigation:

- layered detection
- deterministic rules
- Presidio
- healthcare-specific recognizers
- output inspection
- audit sampling

---

### 2. False Positives

Risk:

    clinically meaningful text is over-redacted.

Mitigation:

- clinical context recognition
- configurable modes
- observe mode
- evaluation datasets
- human review

---

### 3. Utility Loss

Risk:

    excessive anonymization reduces clinical usefulness.

Mitigation:

- semantically preserving replacement
- date shifting instead of deletion
- configurable Safe Harbor vs Expert Determination modes
- utility evaluation metrics

---

### 4. Re-identification Risk

Risk:

    de-identified data can still be linked back to a patient.

Mitigation:

- risk scoring
- suppression/generalization
- k-anonymity or differential privacy in later phases
- periodic re-evaluation

---

### 5. Reversible Mapping Risk

Risk:

    pseudonymization mapping table becomes a sensitive asset.

Mitigation:

- encryption
- key management
- access controls
- audit logs
- limited retention
- separation of duties

---

## Open Questions

1. Should MedMesh implement PHI masking first or full anonymization first?
2. Should PHI governance be built as a new guard or as an extension of PIIScrubberGuard?
3. Should reversible pseudonymization be included in MVP?
4. Should Presidio remain the first backend?
5. Should MedMesh create healthcare-specific Presidio recognizers?
6. How should MedMesh represent Safe Harbor categories in metadata?
7. Should PHI detection run on OUTPUT by default?
8. Should PHI handling depend on model destination?
9. Should PHI decisions be auditable even when no PHI is detected?
10. Should research mode and clinical mode have separate pipeline profiles?

---

## Principal Engineer Recommendation

Do not replace PIIScrubberGuard immediately.

Instead:

1. Keep PIIScrubberGuard as the generic baseline.
2. Add a MedMesh-specific PHI wrapper or subclass.
3. Define metadata conventions first.
4. Add HIPAA Safe Harbor category mapping.
5. Add MedMesh pipeline profiles.
6. Add healthcare-specific recognizers incrementally.
7. Add audit semantics before broadening the PHI system.

The first healthcare-specific implementation should be:

    PHIScrubberGuard

It should build on the existing GuardResult.MODIFY behavior and Presidio backend, while adding MedMesh-specific metadata and healthcare governance modes.

---

## Next Recommended Document

Create:

    docs/architecture/governance-metadata-contracts.md

This document should define standard metadata keys for:

- PHI detection
- PHI transformation
- policy decisions
- FHIR access control
- audit events
- clinical risk classification

