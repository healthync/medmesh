# Security Pipeline Analysis

## Objective

Analyze the existing AgentGuard security subsystem and determine how it can evolve into the MedMesh healthcare AI governance runtime.

---

## Verified Current Structure

Current security subsystem:

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

There is currently no policies, validators, scanners, sandbox, metrics, or standalone monitoring directory under src/agentguard/security.

---

## Verified Security Model

The current security subsystem is a guard-based runtime pipeline.

It is not yet a full healthcare policy engine.

The pipeline executes guards through defined intercept points and should be treated as the current foundation for MedMesh runtime governance.

---

## Current Intercept Points

The known intercept points to verify in code are:

- INPUT
- PRE_LLM
- TOOL_AUTH
- PRE_TOOL
- TOOL_OUTPUT
- OUTPUT
- ASYNC

These appear to be the main extension seams for runtime governance.

---

## Current Guard Categories

### 1. Structural Guards

File: guards/structural.py

Responsibilities to verify:
- input type validation
- input length validation
- encoding validation
- tool argument schema validation

Potential MedMesh evolution:
- FHIR schema validation
- healthcare payload validation
- structured clinical data validation
- terminology payload enforcement

### 2. Pattern Guards

File: guards/pattern.py

Responsibilities to verify:
- prompt injection detection
- jailbreak pattern detection
- dangerous argument detection
- system prompt leakage detection

Potential MedMesh evolution:
- unsafe clinical prompt detection
- healthcare workflow injection detection
- medical instruction manipulation detection
- prompt provenance enforcement

### 3. Budget Guards

File: guards/budget.py

Responsibilities to verify:
- token budgets
- request budgets
- cost budgets
- tool call budgets

Potential MedMesh evolution:
- clinical workflow risk budgets
- high-risk workflow throttling
- expensive model governance
- protected clinical escalation thresholds

### 4. Scope Guards

File: guards/scope.py

Responsibilities to verify:
- tool allowlists
- delegation depth
- human-in-the-loop approval

Potential MedMesh evolution:
- FHIR operation authorization
- patient-scoped authorization
- clinician approval workflows
- high-risk operation escalation
- regulated action enforcement

### 5. Classifier Guards

File: guards/classifiers.py

Responsibilities to verify:
- PII detection
- PII anonymization
- injection classification
- toxicity classification

Potential MedMesh evolution:
- HIPAA Safe Harbor workflows
- healthcare-specific PHI categories
- medical record number handling
- clinical note-aware anonymization
- reversible tokenization
- patient identifier governance

### 6. Semantic Guards

File: guards/semantic.py

Responsibilities to verify:
- semantic prompt injection detection
- LLM-as-judge evaluation

Potential MedMesh evolution:
- unsafe clinical reasoning detection
- hallucination risk scoring
- unsafe recommendation detection
- policy-aware semantic governance

### 7. Monitoring Guards

File: guards/monitors.py

Responsibilities to verify:
- async audit logging
- evaluation sampling
- quality scoring
- cost attribution

Potential MedMesh evolution:
- healthcare audit logging
- PHI leakage telemetry
- clinician review sampling
- governance metrics
- compliance event pipelines

---

## Important Correction

The previous version of this document incorrectly described the security module as having separate folders for:

- policies
- validators
- scanners
- sandbox
- metrics
- monitoring

Those folders have not been verified in the actual codebase and should not be treated as existing architecture.

---

## Current Strengths

### 1. Guard Pipeline Architecture

The current structure suggests a middleware-oriented guard pipeline.

This is valuable because healthcare AI governance requires runtime inspection and enforcement.

### 2. Existing Guard Separation

The current guard files are separated by concern:

- structural validation
- pattern detection
- budget enforcement
- scope enforcement
- classifier-based detection
- semantic analysis
- monitoring

This separation gives MedMesh clear extension points.

### 3. Healthcare Extension Potential

The existing guard model can likely support healthcare-specific extensions without a complete rewrite.

Potential future guards:
- PHIScrubberGuard
- HIPAASafeHarborGuard
- FHIRScopeGuard
- ClinicalOutputSafetyGuard
- PatientContextGuard
- ClinicalAuditGuard

---

## Current Gaps

### 1. No Healthcare PHI Semantics Confirmed Yet

The current classifier layer may support general PII, but healthcare-grade PHI behavior still needs to be verified and likely extended.

Missing or unverified:
- HIPAA Safe Harbor categories
- clinical PHI semantics
- date shifting/generalization
- age over 89 handling
- healthcare identifier classification
- provider/patient identity handling

### 2. No FHIR-Aware Authorization Confirmed Yet

Current scope behavior appears general tool-governance oriented.

Missing or unverified:
- FHIR resource-level permissions
- SMART on FHIR scope mapping
- patient-scoped authorization
- read/write operation control
- sensitivity-aware data governance

### 3. No Dedicated Policy Engine in Security Layer

There is no verified security/policies module.

This means MedMesh may later need:
- centralized healthcare policies
- contextual authorization
- declarative governance rules
- policy composition
- optional OPA/Rego integration

### 4. No Healthcare Audit Semantics Confirmed Yet

Monitoring may exist through guards/monitors.py, but healthcare-specific audit semantics are not yet verified.

Missing or unverified:
- patient context
- accessed FHIR resources
- workflow identity
- clinician identity
- PHI transformation events
- policy decision events
- access justification
- output risk classification

---

## MedMesh Translation

| Current Capability | Future MedMesh Capability |
|---|---|
| Guard Pipeline | MedMesh Governance Runtime |
| Structural Guards | Healthcare Payload Validation |
| Pattern Guards | Clinical Prompt Attack Detection |
| Budget Guards | Resource and Risk Governance |
| Scope Guards | FHIR and Tool Authorization |
| Classifier Guards | PHI and Safety Classification |
| Semantic Guards | Clinical Semantic Safety |
| Monitoring Guards | Healthcare Audit and Compliance Telemetry |

---

## Strategic Direction

MedMesh should not replace the existing guard pipeline immediately.

Recommended path:

1. Verify the core guard contracts.
2. Document actual runtime behavior.
3. Extend the guard model with healthcare-specific guards.
4. Add policy abstraction only after the existing pipeline is understood.
5. Add FHIR-aware authorization after scope guard behavior is verified.
6. Add healthcare audit semantics after monitoring behavior is verified.

---

## Key Future MedMesh Components

### MedMesh PHI Gateway

Responsibilities:
- PHI detection
- healthcare entity recognition
- prompt inspection
- output inspection
- anonymization or masking

### MedMesh Policy Runtime

Responsibilities:
- contextual authorization
- patient-scoped governance
- SMART on FHIR mapping
- healthcare policy enforcement

### MedMesh Clinical Safety Engine

Responsibilities:
- hallucination analysis
- unsafe recommendation detection
- clinical escalation handling
- confidence governance

### MedMesh Audit Engine

Responsibilities:
- audit events
- execution lineage
- policy decision tracing
- PHI transformation logs
- compliance reporting

---

## Recommended Next Analysis Step

Next, inspect and document the actual core contracts:

1. AgentContext
2. GuardResult
3. GuardVerdict
4. GuardMode
5. SecurityPipeline
6. Guard execution flow
7. Async monitoring flow

These define the actual extension model for MedMesh.

