# Guard Implementation Analysis

## Objective

Analyze the existing guard implementations in AgentGuard and determine how they can evolve into MedMesh healthcare AI governance primitives.

This document is based on the verified current security structure:

    src/agentguard/security/
    ├── __init__.py
    ├── pipeline.py
    └── guards/
        ├── __init__.py
        ├── base.py
        ├── budget.py
        ├── classifiers.py
        ├── monitors.py
        ├── pattern.py
        ├── scope.py
        ├── semantic.py
        └── structural.py

Important correction:

The guards are grouped inside module files such as structural.py, pattern.py, budget.py, scope.py, classifiers.py, semantic.py, and monitors.py.

They are not stored as separate files such as input_type_guard.py, input_length_guard.py, or tool_scope_guard.py.

---

## Verified Guard Families

The verified guard families currently present in the repository are:

1. Structural Guards
2. Pattern Guards
3. Budget Guards
4. Scope Guards
5. Classifier Guards
6. Semantic Guards
7. Monitoring Guards

---

## Default Pipeline Status

Based on the verified SecurityPipeline.from_config behavior, the default pipeline instantiates category 1 to category 4 guards.

Enabled by default:

- Structural guards
- Pattern guards
- Budget guards
- Scope guards

Not enabled by default:

- Classifier guards
- Semantic guards
- Monitoring guards

This is an important finding.

It means the current default runtime is primarily a deterministic security and governance pipeline.

For MedMesh, PHI, semantic safety, and audit monitoring must become first-class pipeline profiles rather than optional hidden capabilities.

---

# 1. Structural Guards

Source file:

    src/agentguard/security/guards/structural.py

## Purpose

Structural guards validate basic input and tool payload structure before deeper analysis occurs.

These guards are useful because downstream security checks assume inputs are well-formed.

---

## Current Responsibilities

The structural guard family includes checks for:

- input type
- input length
- encoding safety
- tool argument schema

---

## MedMesh Value

Structural guards are highly reusable for MedMesh.

Healthcare AI systems will receive a mixture of:

- user prompts
- clinical notes
- patient messages
- FHIR JSON payloads
- tool arguments
- retrieved clinical documents
- EHR exports
- OCR-derived text

Basic structural validation is still necessary before PHI detection, FHIR authorization, or clinical safety checks.

---

## Healthcare Extension Opportunities

Future MedMesh structural guards may include:

- FHIRResourceShapeGuard
- ClinicalDocumentShapeGuard
- HL7PayloadShapeGuard
- FHIRBundleSizeGuard
- ClinicalNoteLengthGuard
- TerminologyCodeFormatGuard

---

## Recommendation

Reuse the existing structural guard family.

Do not rewrite it.

Extend it later with healthcare-specific structured payload validation.

---

# 2. Pattern Guards

Source file:

    src/agentguard/security/guards/pattern.py

## Purpose

Pattern guards detect known unsafe strings, phrases, or argument patterns.

---

## Current Responsibilities

The pattern guard family includes detection for:

- prompt injection patterns
- jailbreak phrases
- dangerous argument patterns
- system prompt leakage patterns

---

## MedMesh Value

Pattern guards are highly relevant to healthcare AI because healthcare agents may have access to sensitive clinical data and tools.

Prompt injection in healthcare can attempt to:

- reveal PHI
- bypass consent checks
- ignore safety rules
- expose hidden system prompts
- override clinical restrictions
- perform unauthorized data access
- trick tools into bulk patient retrieval

---

## Healthcare Extension Opportunities

Future MedMesh pattern guards may include:

- ClinicalPromptInjectionGuard
- PHIExfiltrationPatternGuard
- FHIRBulkAccessPatternGuard
- ConsentBypassPatternGuard
- MedicalInstructionOverrideGuard
- HiddenPolicyExtractionGuard

---

## Recommendation

Reuse the existing pattern guard family as part of the MedMesh baseline.

However, do not rely on static patterns alone.

Pattern guards should be layered with:

- scope guards
- PHI guards
- FHIR authorization guards
- output guards
- audit monitoring

---

# 3. Budget Guards

Source file:

    src/agentguard/security/guards/budget.py

## Purpose

Budget guards control usage of tokens, cost, requests, and tool calls.

---

## Current Responsibilities

The budget guard family includes checks for:

- token budget
- cost budget
- request rate
- tool call count

---

## MedMesh Value

Budget guards are reusable and important for operational governance.

In healthcare AI systems, budget controls can also reduce security risk.

Examples:

- limit runaway agents
- limit excessive patient lookups
- prevent recursive tool calls
- reduce denial-of-service risk
- restrict automated data extraction
- limit expensive model usage

---

## Healthcare Extension Opportunities

Future MedMesh budget guards may include:

- PatientLookupRateGuard
- FHIRReadVolumeGuard
- BulkExportThresholdGuard
- HighRiskWorkflowRateGuard
- ClinicalEscalationBudgetGuard
- TenantModelSpendGuard

---

## Recommendation

Reuse the existing budget guard family.

For healthcare, extend budget concepts beyond cost and tokens into data-access volume and clinical workflow risk.

---

# 4. Scope Guards

Source file:

    src/agentguard/security/guards/scope.py

## Purpose

Scope guards restrict what tools or delegated actions an agent can perform.

---

## Current Responsibilities

The scope guard family includes checks for:

- tool allowlists
- delegation depth
- human-in-the-loop gates

---

## MedMesh Value

This is one of the most important guard families for MedMesh.

Healthcare AI agents must operate under least privilege.

Different agents should have different capabilities:

- patient-facing agent
- clinician assistant
- billing assistant
- research agent
- prior authorization agent
- care navigation agent
- data extraction agent

Each agent should have explicit authorization boundaries.

---

## Healthcare Extension Opportunities

Future MedMesh scope guards may include:

- FHIRScopeGuard
- SMARTOnFHIRScopeGuard
- PatientContextGuard
- ConsentContextGuard
- TenantBoundaryGuard
- ClinicianRoleGuard
- ClinicalHumanReviewGuard
- WriteOperationApprovalGuard

---

## Recommendation

Treat the existing scope guard family as a core MedMesh foundation.

This is the starting point for healthcare policy enforcement.

---

# 5. Classifier Guards

Source file:

    src/agentguard/security/guards/classifiers.py

## Purpose

Classifier guards perform model-based or analyzer-based classification of content.

---

## Current Responsibilities

The classifier guard family includes:

- PII detection and anonymization
- injection classification
- toxicity classification

The most important current class for MedMesh is:

    PIIScrubberGuard

---

## PIIScrubberGuard

PIIScrubberGuard uses:

- presidio_analyzer
- presidio_anonymizer

It analyzes text for PII entities and returns MODIFY when entities are detected.

Its metadata includes:

- pii_types
- entity_count

---

## MedMesh Value

This is one of the most strategically important areas for MedMesh.

It can evolve into PHI governance.

Healthcare AI requires PHI controls across:

- user input
- prompts
- tool outputs
- retrieved patient records
- final model output
- audit logs
- research datasets

---

## Current Limitation

PIIScrubberGuard is currently general-purpose PII handling.

It is not yet healthcare-grade PHI governance.

Missing MedMesh-specific capabilities include:

- HIPAA Safe Harbor mapping
- medical record number handling
- health plan beneficiary number handling
- age over 89 handling
- date generalization
- clinical context awareness
- patient-specific PHI policies
- reversible pseudonymization
- PHI audit metadata
- model-destination policy checks

---

## Healthcare Extension Opportunities

Future MedMesh classifier guards may include:

- PHIScrubberGuard
- HIPAASafeHarborGuard
- ClinicalContextPHIGuard
- DateGeneralizationGuard
- AgeGeneralizationGuard
- HealthcareIdentifierGuard
- PHILeakageOutputGuard

---

## Recommendation

Do not replace PIIScrubberGuard immediately.

Use it as the baseline.

Build MedMesh PHI capability as a wrapper, subclass, or new guard that reuses the Presidio foundation while adding healthcare metadata and PHI-specific rules.

---

# 6. Semantic Guards

Source file:

    src/agentguard/security/guards/semantic.py

## Purpose

Semantic guards use deeper classification or LLM-based reasoning to detect unsafe content beyond simple patterns.

---

## Current Status

Semantic guard code exists but is not enabled in the verified default pipeline.

---

## MedMesh Value

Semantic guards may become valuable for healthcare use cases such as:

- unsafe clinical advice detection
- hallucination risk assessment
- subtle PHI leakage detection
- self-harm escalation
- medication advice review
- unsafe diagnosis recommendation detection
- patient-facing output review

---

## Healthcare Risk

Semantic guards introduce additional risks:

- latency
- nondeterminism
- model drift
- explainability limitations
- false positives
- false negatives
- dependency on external LLMs

These risks are especially important in healthcare.

---

## Recommendation

Do not make semantic guards the first line of defense in the MVP.

Use deterministic governance first:

- structural validation
- scope enforcement
- schema validation
- PHI detection
- FHIR authorization
- output inspection

Then add semantic guards as a secondary safety layer.

---

# 7. Monitoring Guards

Source file:

    src/agentguard/security/guards/monitors.py

## Purpose

Monitoring guards support non-blocking observation, audit, telemetry, evaluation, or quality checks.

---

## Current Status

Monitoring guard code exists but is not enabled in the verified default pipeline.

---

## MedMesh Value

Monitoring is critical for healthcare governance.

Healthcare AI systems need strong auditability.

Monitoring guards can support:

- audit logging
- PHI event capture
- policy decision tracing
- clinical safety sampling
- latency metrics
- model output evaluation
- compliance reporting
- incident investigation

---

## Healthcare Extension Opportunities

Future MedMesh monitoring guards may include:

- HealthcareAuditGuard
- PHIAuditGuard
- FHIRAccessAuditGuard
- PolicyDecisionAuditGuard
- ClinicalQualitySamplingGuard
- PHILeakageMonitor
- GovernanceTelemetryGuard

---

## Recommendation

Monitoring must become first-class in MedMesh.

However, first define metadata contracts and audit event shape.

Then wire monitoring guards into MedMesh-specific pipeline profiles.

---

# Reusability Matrix

| Guard Family | Reusable For MedMesh | Refactor Needed | Healthcare Importance |
|---|---|---|---|
| Structural | Yes | Low | High |
| Pattern | Yes | Medium | High |
| Budget | Yes | Low to Medium | Medium |
| Scope | Yes | Medium to High | Critical |
| Classifier | Partial | High | Critical |
| Semantic | Partial | Medium to High | High, later phase |
| Monitoring | Partial | Medium | Critical |

---

# Highest-Value Existing Guard Foundations

The most valuable existing foundations for MedMesh are:

1. Scope guard family
2. Structural guard family
3. Pattern guard family
4. PIIScrubberGuard
5. Budget guard family
6. Monitoring guard family
7. Semantic guard family

---

# Highest-Priority New MedMesh Guards

The highest-priority healthcare-specific guards are:

1. PHIScrubberGuard
2. HIPAASafeHarborGuard
3. FHIRScopeGuard
4. PatientContextGuard
5. ConsentContextGuard
6. HealthcareAuditGuard
7. PHILeakageOutputGuard
8. ClinicalHumanReviewGuard
9. ClinicalOutputSafetyGuard

---

# Recommended Guard Evolution Strategy

## Phase 1: Preserve Existing Guard Pipeline

Do not rewrite the current architecture.

The existing guard-based runtime is useful.

---

## Phase 2: Define MedMesh Pipeline Profiles

Create healthcare-specific profiles such as:

- medmesh-dev
- medmesh-clinical-internal
- medmesh-external-model-strict
- medmesh-research
- medmesh-patient-facing

Each profile should explicitly define:

- enabled guards
- intercept points
- enforcement modes
- PHI handling mode
- audit requirements

---

## Phase 3: Add PHI Governance

Implement:

- PHIScrubberGuard
- Safe Harbor metadata mapping
- OUTPUT PHI leakage inspection
- audit metadata enrichment

---

## Phase 4: Add FHIR Authorization

Implement:

- FHIRScopeGuard
- PatientContextGuard
- SMART scope validation
- FHIR operation metadata

---

## Phase 5: Add Audit Monitoring

Implement:

- HealthcareAuditGuard
- PHIAuditGuard
- FHIRAccessAuditGuard
- PolicyDecisionAuditGuard

---

## Phase 6: Add Semantic Clinical Safety

Implement later:

- ClinicalOutputSafetyGuard
- UnsafeClinicalAdviceGuard
- HallucinationRiskGuard
- HumanReviewRecommendationGuard

---

# Important Design Rule

For the MVP, MedMesh should prioritize deterministic governance before semantic governance.

Order of importance:

1. deterministic policy and scope enforcement
2. PHI detection and transformation
3. FHIR authorization
4. auditability
5. semantic clinical safety

This keeps the first version understandable, testable, and auditable.

---

# Principal Engineer Recommendation

The existing guard implementation structure is usable.

Do not rename or restructure the guards prematurely.

Instead:

- document the existing guard families
- introduce MedMesh-specific guards beside existing guards
- add healthcare pipeline profiles
- preserve fail-closed behavior
- standardize metadata
- add healthcare auditability

The next technical design step should focus on MedMesh pipeline profiles.

---

# Next Recommended Document

Create:

    docs/architecture/medmesh-pipeline-profiles.md

This document should define the runtime profiles that decide which guards are enabled for different healthcare AI deployment scenarios.

