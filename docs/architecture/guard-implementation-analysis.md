# Guard Implementation Analysis

## Objective

Analyze the existing guard implementations in AgentGuard and determine:

- which guards are reusable for MedMesh
- which guards require healthcare-specific extensions
- which guards are not suitable for healthcare AI governance
- where new healthcare-native guards must be introduced

This document focuses on concrete implementation behavior rather than high-level runtime contracts.

---

## Verified Guard Families

The verified guard families currently present in the repository are:

1. Structural Guards
2. Pattern Guards
3. Budget Guards
4. Scope Guards
5. Semantic Guards
6. PII Guards
7. Monitoring Guards

However, only some of these are enabled by default in SecurityPipeline.from_config().

---

# 1. Structural Guards

Verified files:

- input_type_guard.py
- input_length_guard.py
- encoding_guard.py
- tool_argument_schema_guard.py

---

## 1.1 InputTypeGuard

### Purpose

Validates that the incoming payload is a string.

### Current Behavior

Passes:
- valid string input

Blocks:
- non-string payloads

### Security Value

Protects downstream guards from invalid assumptions about payload shape.

### MedMesh Assessment

Still useful for healthcare systems.

Healthcare AI pipelines will receive:

- prompts
- transcribed notes
- summaries
- patient messages
- clinician instructions

String validation remains relevant.

### Limitation

Healthcare workflows may eventually require structured payload support:

- FHIR JSON
- CDA/XML
- HL7 payloads
- structured clinical events

### Recommendation

Keep for MVP.

Later extend with structured payload validation.

Potential future variants:

- FHIRResourceTypeGuard
- ClinicalDocumentSchemaGuard
- HL7PayloadGuard

---

## 1.2 InputLengthGuard

### Purpose

Blocks excessively large payloads.

### Current Behavior

Checks character length against configured threshold.

### Security Value

Protects against:

- prompt flooding
- memory exhaustion
- oversized payload abuse

### MedMesh Assessment

Strongly reusable.

Healthcare environments contain:

- very long clinical notes
- discharge summaries
- pathology reports
- radiology interpretations

Length control remains important.

### Healthcare-Specific Considerations

Healthcare payloads may legitimately exceed ordinary LLM prompt sizes.

Need differentiated thresholds for:

- patient messages
- clinician notes
- FHIR bundles
- imaging reports

### Recommendation

Reuse directly for MVP.

Later add contextual thresholding.

---

## 1.3 EncodingGuard

### Purpose

Detects invalid or suspicious encoding patterns.

### Current Behavior

Blocks malformed encoding and potentially dangerous character sequences.

### Security Value

Protects against:

- encoding abuse
- parser confusion
- hidden payload injection

### MedMesh Assessment

Strongly reusable.

Healthcare systems frequently exchange:

- OCR text
- copied EHR exports
- external PDFs
- multilingual patient content

Encoding normalization remains important.

### Recommendation

Reuse directly with minimal modification.

---

## 1.4 ToolArgumentSchemaGuard

### Purpose

Validates tool arguments against expected schema structure.

### Current Behavior

Ensures tool calls match expected parameter structure.

### Security Value

Protects tool layer from malformed arguments.

### MedMesh Assessment

Extremely important for healthcare.

Healthcare tools may include:

- FHIR readers
- medication systems
- scheduling APIs
- patient search
- CDS services
- EHR write operations

Schema validation is foundational.

### Major Healthcare Extension Opportunity

Future healthcare schemas may validate:

- patient_id format
- encounter_id format
- FHIR resource structure
- restricted operation types
- allowed search scopes

### Recommendation

One of the highest-value reusable guards.

Should become a core MedMesh foundation.

---

# 2. Pattern Guards

Verified files:

- injection_pattern_guard.py
- jailbreak_phrase_guard.py
- dangerous_argument_guard.py

---

## 2.1 InjectionPatternGuard

### Purpose

Detects known prompt injection patterns.

### Current Behavior

Uses regex/pattern matching for suspicious prompt structures.

### Security Value

Protects against:

- instruction override
- hidden system prompt extraction
- prompt hijacking

### MedMesh Assessment

Critical for healthcare AI.

Healthcare environments are high-risk because prompt injection could expose:

- PHI
- treatment logic
- internal prompts
- protected records

### Healthcare-Specific Risks

Potential attacks:

- reveal all patient records
- ignore HIPAA restrictions
- bypass consent checks
- expose hidden policies
- override clinical safeguards

### Recommendation

Strongly reusable.

Should become mandatory in MedMesh.

---

## 2.2 JailbreakPhraseGuard

### Purpose

Detects known jailbreak phrases and override attempts.

### Current Behavior

Uses phrase matching.

### Security Value

Blocks common adversarial prompts.

### MedMesh Assessment

Important but insufficient alone.

Healthcare attacks may be more subtle than consumer jailbreak attempts.

### Limitation

Static phrase matching has limited long-term reliability.

### Recommendation

Keep for layered defense.

Do not rely on it alone.

Eventually complement with semantic detection.

---

## 2.3 DangerousArgumentGuard

### Purpose

Detects dangerous tool argument patterns.

### Current Behavior

Blocks suspicious argument values.

### Security Value

Protects tools from unsafe invocation.

### MedMesh Assessment

Very important for healthcare.

Potential healthcare abuse examples:

- bulk patient export
- unrestricted patient search
- unauthorized writes
- unsafe medication changes
- excessive query scope

### Recommendation

One of the most important reusable guards.

Should evolve into healthcare-aware policy validation.

Potential future versions:

- FHIRDangerousOperationGuard
- ClinicalWriteProtectionGuard
- PatientScopeRestrictionGuard

---

# 3. Budget Guards

Verified files:

- token_budget_guard.py
- cost_budget_guard.py
- request_rate_guard.py
- tool_call_count_guard.py

---

## 3.1 TokenBudgetGuard

### Purpose

Enforces token consumption limits.

### Current Behavior

Blocks requests exceeding token budget.

### Security Value

Protects against runaway usage.

### MedMesh Assessment

Useful operationally.

Not healthcare-specific but still important.

### Recommendation

Reuse directly.

---

## 3.2 CostBudgetGuard

### Purpose

Enforces cost limits.

### Current Behavior

Blocks requests exceeding cost threshold.

### MedMesh Assessment

Operationally useful.

May become important for enterprise healthcare tenants.

### Recommendation

Reuse directly.

---

## 3.3 RequestRateGuard

### Purpose

Limits request frequency.

### Current Behavior

Blocks excessive request rate.

### Security Value

Protects against abuse and flooding.

### MedMesh Assessment

Very important for healthcare APIs.

Potential threats:

- patient enumeration
- scraping
- automated extraction
- denial-of-service behavior

### Recommendation

Strongly reusable.

---

## 3.4 ToolCallCountGuard

### Purpose

Limits number of tool calls per execution.

### Current Behavior

Blocks excessive tool chaining.

### Security Value

Prevents runaway agent behavior.

### MedMesh Assessment

Important for healthcare agents.

Potential abuse:

- mass patient lookup
- recursive retrieval
- automated bulk extraction

### Recommendation

Strongly reusable.

---

# 4. Scope Guards

Verified file:

- tool_scope_guard.py

---

## 4.1 ToolScopeGuard

### Purpose

Restricts which tools an agent can invoke.

### Current Behavior

Checks tool name against AgentContext.allowed_tools.

### Security Value

Provides least-privilege enforcement.

### MedMesh Assessment

One of the most important guards in the entire system.

This maps directly to healthcare authorization.

### Potential Healthcare Mapping

Examples:

- clinician agents
- billing agents
- triage agents
- patient-facing assistants
- research agents

Each may require different access boundaries.

### Future Healthcare Extensions

Potential future guards:

- FHIRScopeGuard
- SMARTScopeGuard
- PatientConsentGuard
- TenantBoundaryGuard

### Recommendation

This should become a cornerstone MedMesh governance primitive.

---

# 5. Semantic Guards

Verified files exist but are not enabled by default.

Examples include:

- semantic_classifier_guard.py
- semantic_policy_guard.py

---

## Current Assessment

The semantic guard layer appears intended for ML-based classification or policy reasoning.

However, it is not wired into the default pipeline.

### Potential MedMesh Value

Very high.

Potential healthcare use cases:

- unsafe clinical advice detection
- suicide/self-harm detection
- treatment hallucination detection
- PHI leakage classification
- inappropriate patient guidance
- high-risk workflow escalation

### Current Risk

Semantic systems introduce:

- latency
- nondeterminism
- model drift
- explainability concerns

These are especially important in healthcare.

### Recommendation

Do not rely heavily on semantic guards in the first MedMesh MVP.

Use deterministic controls first:

- scope
- schema
- policy
- PHI rules
- structural validation

Add semantic governance later as a secondary layer.

---

# 6. PII Guards

Verified files exist but are not enabled by default.

Examples include:

- pii_guard.py
- pii_redaction_guard.py

---

## Current Assessment

This is one of the most strategically important areas for MedMesh.

### Potential Healthcare Value

Extremely high.

Potential use cases:

- PHI detection
- HIPAA Safe Harbor enforcement
- de-identification
- anonymization
- tokenization
- output filtering

### Current Unknowns

The implementation quality still requires deeper analysis:

- regex-only?
- NER-based?
- deterministic?
- confidence thresholds?
- false-positive handling?
- healthcare entity support?

### Recommendation

This should become a primary MedMesh workstream.

Need deeper analysis next.

Recommended future document:

    pii-and-phi-governance-analysis.md

---

# 7. Monitoring Guards

Verified monitor guard files exist but are not enabled by default.

Examples:

- telemetry_guard.py
- audit_guard.py
- monitor_guard.py

---

## Current Assessment

Potentially valuable for healthcare observability.

### Potential Healthcare Use Cases

- audit logging
- governance telemetry
- policy metrics
- PHI access tracing
- compliance evidence
- clinical workflow monitoring

### Healthcare Importance

Healthcare governance requires auditability.

This area may become foundational.

### Recommendation

Very important for enterprise MedMesh.

Need detailed analysis later.

---

# Important Architectural Finding

Only a subset of guards are enabled in the default pipeline.

Currently enabled by default:

- structural guards
- pattern guards
- budget guards
- scope guards

Not enabled by default:

- semantic guards
- PII guards
- monitoring guards

This means the repository currently behaves more like a secure AI runtime than a healthcare governance runtime.

That is acceptable for the current architecture stage.

---

# Strongest Reusable Foundations For MedMesh

The highest-value reusable guards are:

1. ToolScopeGuard
2. ToolArgumentSchemaGuard
3. DangerousArgumentGuard
4. InjectionPatternGuard
5. RequestRateGuard
6. ToolCallCountGuard
7. InputLengthGuard

These provide strong deterministic governance foundations.

---

# Highest-Priority Healthcare Extensions

The most important new healthcare-specific guard areas are:

1. PHI Detection
2. PHI Redaction
3. FHIR Authorization
4. Consent Enforcement
5. Clinical Safety Review
6. Audit Logging
7. Patient Boundary Enforcement

---

# Recommended MedMesh Guard Strategy

## Phase 1

Reuse deterministic infrastructure:

- scope guards
- schema guards
- pattern guards
- budget guards

## Phase 2

Introduce healthcare deterministic controls:

- PHI guards
- FHIR scope guards
- consent guards
- patient boundary guards

## Phase 3

Introduce semantic healthcare controls:

- hallucination detection
- unsafe clinical advice detection
- escalation classification

## Phase 4

Introduce enterprise governance:

- policy orchestration
- tenant governance
- audit analytics
- compliance dashboards

---

# Principal Engineer Recommendation

Do not attempt to replace the current guard architecture.

Instead:

- preserve the deterministic pipeline
- treat healthcare governance as additional layered guards
- keep semantic systems secondary initially
- prioritize auditability and explainability
- preserve fail-closed behavior

The current architecture is a strong foundation for MedMesh if healthcare-specific governance layers are added carefully.

---

# Recommended Next Analysis

Next recommended document:

    docs/architecture/pii-and-phi-governance-analysis.md

This next document should deeply analyze:

- current PII guard implementations
- PHI detection requirements
- HIPAA Safe Harbor requirements
- de-identification strategies
- healthcare NER approaches
- tokenization vs masking
- reversible anonymization
- audit implications

