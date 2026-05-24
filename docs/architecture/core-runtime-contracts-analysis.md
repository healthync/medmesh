# Core Runtime Contracts Analysis

## Objective

Analyze the foundational runtime contracts in the current AgentGuard codebase and determine how they can support the future MedMesh healthcare AI governance runtime.

This document is based on the verified files:

- src/agentguard/core/context.py
- src/agentguard/core/verdict.py
- src/agentguard/security/guards/base.py
- src/agentguard/security/pipeline.py

---

## Why This Matters

MedMesh is intended to become the governance mesh for healthcare AI agents.

To support that goal, the runtime must provide reliable contracts for:

- request context propagation
- guard execution
- governance decisions
- content modification
- blocking behavior
- tool authorization
- async monitoring
- auditability
- future healthcare-specific metadata

The core runtime contracts determine whether MedMesh can evolve from the existing AgentGuard architecture without a full rewrite.

---

## Verified Core Contracts

The verified runtime contracts are:

1. AgentContext
2. GuardVerdict
3. GuardResult
4. GuardMode
5. InterceptPoint
6. SecurityGuard
7. SecurityPipeline
8. PipelineResult

---

## 1. AgentContext

Source:

    src/agentguard/core/context.py

### Verified Design

AgentContext is declared as an immutable dataclass:

    @dataclass(frozen=True)

This means the context is not mutated in place.

When runtime state changes, a new AgentContext is created and returned.

The code comments describe AgentContext as:

    immutable per-request context threaded through every pipeline call

### Current Fields

AgentContext currently contains:

- session_id
- task_id
- turn_number
- agent_identity
- allowed_tools
- token_budget_remaining
- cost_budget_remaining_usd
- tool_call_count
- llm_call_count

### Current Validation

AgentContext validates that:

- session_id cannot be empty
- task_id cannot be empty

### Current Helper Methods

AgentContext currently supports:

- is_tool_allowed(tool_name)
- has_token_budget(tokens_needed)
- has_cost_budget(cost_usd)
- after_llm_call(tokens_used, cost_usd)
- after_tool_call()
- for_sub_agent(...)
- for_testing(...)

### Runtime Meaning

AgentContext is the central execution context passed into guards, instrumentors, and loggers.

It currently models:

- identity
- task/session tracking
- tool permissions
- token budget
- cost budget
- tool call count
- LLM call count
- sub-agent delegation context

### MedMesh Assessment

AgentContext is a strong foundation because it is:

- immutable
- explicit
- passed through guard execution
- capable of modeling sub-agent delegation
- already connected to tool authorization and budgets

However, it is not yet healthcare-aware.

### Healthcare Gaps

AgentContext does not currently include:

- patient_id
- encounter_id
- clinician_id
- organization_id
- tenant_id
- consent_context
- FHIR scopes
- SMART on FHIR context
- accessed FHIR resources
- PHI classification state
- audit correlation ID
- workflow ID
- clinical risk level

### Future MedMesh Direction

MedMesh should eventually extend the context model to support healthcare governance metadata.

Potential future approach:

- avoid modifying AgentContext too early
- first introduce a healthcare_context field or wrapper
- later decide whether MedMesh needs a dedicated MedMeshContext
- preserve immutability for auditability and predictability

Potential future healthcare fields:

- patient_id
- encounter_id
- clinician_id
- organization_id
- tenant_id
- workflow_id
- consent_context
- fhir_scopes
- smart_context
- accessed_resources
- phi_entities
- policy_decisions
- audit_events
- risk_scores

### Important Design Consideration

Because AgentContext is frozen, MedMesh must avoid mutation-based governance state.

Instead, MedMesh should use one of these patterns:

1. return a new context when governance state changes
2. attach governance data to GuardResult.metadata
3. introduce a separate immutable HealthcareContext
4. introduce a MedMeshExecutionContext that composes AgentContext and HealthcareContext

This is a future architectural decision.

---

## 2. GuardVerdict

Source:

    src/agentguard/core/verdict.py

### Verified Verdicts

The current GuardVerdict enum contains only:

- PASS
- BLOCK
- MODIFY

### Meaning

PASS means the guard allows execution to continue.

BLOCK means the guard stops the current execution path.

MODIFY means the guard transforms the input and allows execution to continue with modified content.

### MedMesh Assessment

The current verdict model is simple and useful.

It supports the minimum runtime governance needs:

- allow
- deny
- transform

This is enough for the current system and enough for an initial MedMesh PHI gateway.

### Current Limitation

The verdict model does not currently include:

- ESCALATE
- REQUIRE_REVIEW
- ALLOW_WITH_AUDIT
- QUARANTINE
- DEFER
- WARN

These should not be treated as existing behavior.

They are potential future MedMesh extensions.

### Future MedMesh Direction

For the MVP, MedMesh should probably keep the existing verdict model.

Reason:

- PASS supports normal flow
- BLOCK supports hard governance enforcement
- MODIFY supports PHI masking and transformation

Future verdict extensions should be considered only after the healthcare runtime needs are proven.

Potential future verdicts:

- REQUIRE_REVIEW for clinician approval
- ESCALATE for high-risk workflows
- ALLOW_WITH_AUDIT for permitted but sensitive workflows
- QUARANTINE for unsafe or suspicious payloads

### Recommended Decision For Now

Do not modify GuardVerdict yet.

Use existing PASS, BLOCK, and MODIFY to implement initial MedMesh behavior.

---

## 3. GuardResult

Source:

    src/agentguard/core/verdict.py

### Verified Design

GuardResult is declared as:

    @dataclass(frozen=True)

This means guard results are immutable.

### Current Fields

GuardResult currently contains:

- verdict
- guard_name
- latency_ms
- modified_input
- reason
- metadata

### Metadata Behavior

GuardResult converts metadata into an immutable mapping using MappingProxyType.

This prevents guards from mutating each other's result metadata after creation.

### Factory Methods

GuardResult currently provides:

- pass_(guard_name, latency_ms)
- block(guard_name, latency_ms, reason, metadata)
- modify(guard_name, latency_ms, modified_input, reason, metadata)

### Convenience Properties

GuardResult currently provides:

- is_pass
- is_block
- is_modify

### Runtime Meaning

GuardResult is the unit of governance decision returned by every guard.

It carries:

- what decision was made
- which guard made it
- how long it took
- why it happened
- what metadata was attached
- whether content was transformed

### MedMesh Assessment

GuardResult is a strong audit foundation.

It can support healthcare governance because metadata can carry additional decision details.

Potential MedMesh metadata examples:

- phi_entities_detected
- phi_categories
- anonymization_method
- fhir_resource_type
- fhir_operation
- policy_id
- policy_decision
- patient_context_present
- clinical_risk_score
- escalation_reason
- audit_event_id

### Current Limitation

GuardResult.metadata is generic and not typed.

This gives flexibility, but it can become inconsistent if not governed.

### Future MedMesh Direction

For MVP, use metadata carefully with documented keys.

Later, define typed metadata contracts such as:

- PHIGuardMetadata
- FHIRAuthorizationMetadata
- ClinicalSafetyMetadata
- AuditMetadata
- PolicyDecisionMetadata

### Recommended Decision For Now

Use GuardResult.metadata as the first extension point for healthcare governance, but document all metadata keys.

Do not introduce typed metadata until at least two or three MedMesh healthcare guards exist.

---

## 4. GuardMode

Source:

    src/agentguard/security/guards/base.py

### Verified Modes

GuardMode currently contains:

- ENFORCE
- OBSERVE
- DISABLED

### Current Behavior

DISABLED mode immediately returns PASS.

ENFORCE mode applies guard results normally.

OBSERVE mode converts BLOCK to PASS.

### Important Detail

In OBSERVE mode, a BLOCK result is converted to PASS.

However, based on the verified base guard behavior, the returned PASS result does not preserve the original block reason or metadata.

### Runtime Meaning

GuardMode allows staged rollout of governance rules.

This is important for enterprise deployment.

### MedMesh Assessment

GuardMode is highly valuable for healthcare.

Healthcare AI governance should rarely start in hard enforcement mode.

A safer rollout pattern is:

1. DISABLED during development
2. OBSERVE during evaluation
3. ENFORCE after false-positive analysis

### Healthcare Use Cases

Potential MedMesh usage:

- observe PHI leakage before blocking
- observe FHIR authorization failures before enforcement
- observe unsafe clinical outputs before blocking
- compare guard outcomes across environments
- tune rules with real-world data

### Current Limitation

OBSERVE mode may lose important information about what would have been blocked.

For healthcare, that information is valuable.

### Future MedMesh Direction

MedMesh should eventually preserve would-have-blocked metadata in OBSERVE mode.

Possible future metadata:

- observed_verdict: block
- observed_reason
- observed_guard
- observed_metadata
- enforcement_mode: observe

### Recommended Decision For Now

Do not change GuardMode yet.

Document the limitation and revisit when building healthcare audit semantics.

---

## 5. InterceptPoint

Source:

    src/agentguard/security/guards/base.py

### Verified Intercept Points

InterceptPoint currently contains:

- INPUT
- PRE_LLM
- TOOL_AUTH
- PRE_TOOL
- TOOL_OUTPUT
- OUTPUT
- ASYNC

### Runtime Meaning

Intercept points define where guards attach in the AI execution lifecycle.

### Current Use

The SecurityPipeline buckets guards by intercept point.

At runtime, only guards relevant to a specific intercept point are executed.

### MedMesh Assessment

InterceptPoint is one of the strongest extension seams.

It maps naturally to healthcare AI governance.

### MedMesh Mapping

INPUT:
- inspect user input
- detect PHI
- detect unsafe clinical task requests
- validate request format

PRE_LLM:
- inspect final prompt before model call
- remove or tokenize PHI
- enforce prompt policy
- apply model-routing governance

TOOL_AUTH:
- authorize tool call
- enforce tool allowlist
- enforce FHIR access control
- enforce patient-scoped permissions

PRE_TOOL:
- inspect tool arguments
- validate FHIR operation
- enforce write restrictions
- block unsafe clinical actions

TOOL_OUTPUT:
- inspect data returned from tools
- detect PHI
- classify resource sensitivity
- restrict leakage into model context

OUTPUT:
- inspect final model output
- detect PHI leakage
- detect unsafe clinical recommendation
- enforce patient-facing communication rules

ASYNC:
- write audit events
- sample for evaluation
- monitor quality
- collect governance telemetry

### Current Limitation

The current intercept point model is generic and not healthcare-specific.

That is acceptable.

MedMesh should first use the existing intercept points before adding new ones.

### Potential Future Healthcare Intercept Points

Only if needed later:

- PRE_FHIR
- POST_FHIR
- PRE_PATIENT_OUTPUT
- CLINICIAN_REVIEW
- CONSENT_CHECK

### Recommended Decision For Now

Keep the existing intercept points.

Map healthcare governance onto them.

Avoid adding healthcare-specific intercept points until the need becomes clear.

---

## 6. SecurityGuard

Source:

    src/agentguard/security/guards/base.py

### Verified Design

SecurityGuard is the abstract base class for all guards.

Subclasses can implement:

- check(input, context)
- check_tool(tool_name, args, context)

The base class provides wrappers:

- run(input, context)
- run_tool(tool_name, args, context)

### Verified Class Attributes

Subclasses can define:

- category
- default_intercept_points
- default_timeout_ms

### Verified Runtime Behavior

SecurityGuard provides:

- setup()
- teardown()
- health_check()
- check()
- check_tool()
- run()
- run_tool()
- _timed_call()
- _apply_mode()
- run_contract_test()

### Fail-Closed Behavior

SecurityGuard blocks when:

- a guard raises an exception
- a guard exceeds timeout

This is important for safety.

### Timeout Behavior

If guard latency exceeds timeout_ms, SecurityGuard returns BLOCK.

### Exception Behavior

If a guard raises an exception, SecurityGuard returns BLOCK.

### Observe Mode Behavior

If mode is OBSERVE and a guard returns BLOCK, SecurityGuard converts the result to PASS.

### Contract Testing

SecurityGuard includes run_contract_test().

It verifies:

- clean input must pass
- empty input must not raise
- health_check returns bool

### MedMesh Assessment

SecurityGuard is a strong extension base for healthcare-specific guards.

Potential future guards:

- PHIScrubberGuard
- HIPAASafeHarborGuard
- ClinicalNotePHIGuard
- PatientContextGuard
- FHIRScopeGuard
- FHIRWriteBlockGuard
- ClinicalOutputSafetyGuard
- ClinicalHumanReviewGuard
- HealthcareAuditGuard

### Current Limitation

SecurityGuard only accepts string input for content checks.

That may limit structured healthcare payload handling.

FHIR resources, clinical notes, and tool outputs may need richer payload structures.

### Future MedMesh Direction

For the MVP, serialize structured payloads to strings where needed.

Later, consider richer payload contracts:

- GuardPayload
- ToolPayload
- FHIRPayload
- ClinicalDocumentPayload

### Recommended Decision For Now

Do not change SecurityGuard yet.

Build MedMesh healthcare guards within the current contract first.

---

## 7. SecurityPipeline

Source:

    src/agentguard/security/pipeline.py

### Verified Design

SecurityPipeline orchestrates guards across intercept points.

### Initialization Behavior

During construction:

- guards are sorted by category
- guards are bucketed by intercept point
- async guards are separated into _async_guards

### Guard Ordering

Guards run in category order.

This is important because cheaper or simpler guards can run before expensive guards.

### Content Execution

SecurityPipeline.run(input, context, intercept_point) executes content guards for a given intercept point.

Execution behavior:

1. get guards for intercept point
2. execute guards sequentially
3. collect GuardResult objects
4. stop immediately if a guard blocks
5. update current_input if a guard modifies input
6. return final PipelineResult

### Tool Execution

SecurityPipeline.run_tool(tool_name, args, context, intercept_point) executes tool guards.

Execution behavior:

1. get guards for tool intercept point
2. execute guards sequentially using run_tool
3. stop immediately if a guard blocks
4. return PipelineResult

### Important Tool Limitation

Tool guard execution currently does not propagate modified args.

Even if a tool guard returned MODIFY, run_tool currently does not update current_args.

This may be acceptable for current usage, but it matters for MedMesh.

### Async Execution

SecurityPipeline.run_async_tail(input, context) runs async guards after the main path.

If an event loop exists, async guards are scheduled as a task.

If no event loop exists, async guards run synchronously.

Async guard exceptions are swallowed.

### Default Pipeline Construction

SecurityPipeline.from_config() currently creates default guards from these categories:

Category 1:
- InputTypeGuard
- InputLengthGuard
- EncodingGuard
- ToolArgumentSchemaGuard

Category 2:
- InjectionPatternGuard
- JailbreakPhraseGuard
- DangerousArgumentGuard

Category 3:
- TokenBudgetGuard
- CostBudgetGuard
- RequestRateGuard
- ToolCallCountGuard

Category 4:
- ToolScopeGuard

### Important Finding

Although classifier, semantic, and monitor guard files exist, they are not instantiated in the verified default from_config() pipeline.

This means:

- PII scrubbing is not enabled by default
- semantic guard behavior is not enabled by default
- async monitoring guards are not enabled by default

### MedMesh Assessment

SecurityPipeline is a strong foundation for MedMesh.

It already supports:

- ordered guard execution
- short-circuit blocking
- content modification
- tool authorization
- async monitoring
- health checks
- setup and teardown lifecycle

### Current Gaps

SecurityPipeline does not yet provide:

- typed healthcare context
- typed policy decisions
- FHIR-aware authorization
- structured tool argument modification
- persistent audit event model
- healthcare-specific telemetry
- full async monitoring by default

### Recommended Direction

Do not rewrite SecurityPipeline now.

First, extend it through new guards and configuration.

Later, consider enhancements such as:

- propagating modified tool arguments
- preserving OBSERVE metadata
- adding typed audit hooks
- adding healthcare-specific pipeline profiles
- enabling PHI and monitoring guards through config

---

## 8. PipelineResult

Source:

    src/agentguard/security/pipeline.py

### Verified Fields

PipelineResult contains:

- verdict
- intercept_point
- guard_results
- final_input
- latency_ms

### Verified Properties

PipelineResult provides:

- is_blocked
- blocking_guard

### Runtime Meaning

PipelineResult is the full result of running a set of guards at an intercept point.

It contains:

- final pipeline verdict
- all guard results
- final transformed input
- total latency
- blocking guard if one exists

### MedMesh Assessment

PipelineResult is useful for healthcare auditability.

Potential MedMesh usage:

- store full guard result chain
- record PHI transformations
- record policy enforcement outcomes
- record latency of governance path
- correlate guard events to clinical workflows

### Current Limitation

PipelineResult does not currently include:

- audit_event_id
- trace_id
- patient_id
- policy_bundle_version
- execution_id
- risk_summary

These can be added later through metadata or a higher-level MedMesh wrapper.

### Recommended Decision For Now

Do not modify PipelineResult yet.

Use it as the current runtime decision record and wrap it later if needed.

---

## Core Runtime Strengths

### 1. Immutable Contracts

AgentContext and GuardResult are immutable.

This is very useful for:

- predictability
- safer concurrent execution
- audit reliability
- preventing hidden side effects

### 2. Simple Verdict Model

PASS, BLOCK, and MODIFY provide a clean foundation.

This is enough for early MedMesh governance behavior.

### 3. Guard-Based Extensibility

SecurityGuard gives MedMesh a natural extension mechanism.

Healthcare governance can initially be implemented as new guards.

### 4. Intercept Point Architecture

Intercept points map cleanly to AI execution stages.

This is one of the strongest architectural assets.

### 5. Fail-Closed Safety

Guard errors and guard timeouts block by default.

This is appropriate for safety-sensitive healthcare environments.

### 6. Category-Based Ordering

Guard category ordering allows cheaper checks to run before expensive checks.

This supports efficient governance pipelines.

### 7. Async Tail Support

Async guards can support audit and monitoring without blocking the main path.

---

## Core Runtime Gaps

### 1. No Healthcare Context

The runtime does not yet model:

- patient
- clinician
- encounter
- tenant
- consent
- FHIR scopes
- healthcare workflow

### 2. Generic Metadata

GuardResult.metadata is flexible but untyped.

This is useful early, but could become inconsistent later.

### 3. Limited Verdict Semantics

There is no native escalation or review verdict yet.

### 4. Tool Argument Modification Gap

Content checks support modification propagation.

Tool checks do not currently propagate modified arguments.

### 5. Observe Mode Loses Block Details

OBSERVE mode converts BLOCK to PASS without preserving full would-have-blocked details.

This matters for healthcare governance evaluation.

### 6. PII and Semantic Guards Not Enabled By Default

The default pipeline does not currently instantiate classifier, semantic, or monitor guards.

This matters because MedMesh will need PHI and monitoring to be first-class.

---

## MedMesh Compatibility Assessment

The current runtime is compatible with MedMesh as a foundation.

Compatibility level:

    Strong foundation, healthcare extensions required.

The architecture does not need to be rewritten immediately.

The better path is:

1. preserve existing runtime contracts
2. add healthcare-aware guards
3. define healthcare metadata conventions
4. add MedMesh configuration profiles
5. later introduce typed healthcare context if necessary

---

## Recommended Near-Term MedMesh Plan

### Phase 1: Document Existing Runtime

Complete documentation for:

- core contracts
- security pipeline
- guard categories
- default configuration
- current gaps

### Phase 2: Add Healthcare Guard Concepts

Design, but do not immediately implement:

- PHIScrubberGuard
- HIPAASafeHarborGuard
- FHIRScopeGuard
- PatientContextGuard
- ClinicalOutputSafetyGuard
- HealthcareAuditGuard

### Phase 3: Define Healthcare Metadata Keys

Create a metadata convention document for:

- PHI detection
- policy decisions
- FHIR authorization
- audit events
- clinical risk scoring

### Phase 4: Define MedMesh Runtime Context Strategy

Decide whether to use:

- AgentContext extension
- HealthcareContext wrapper
- MedMeshExecutionContext
- GuardResult metadata only

### Phase 5: Implement MVP Profile

Create a MedMesh MVP pipeline profile using:

- existing structural guards
- existing pattern guards
- existing budget guards
- existing scope guard
- new PHI guard
- new healthcare audit guard

---

## Key Architectural Decisions Not Yet Made

The following decisions should not be made until more analysis is complete:

1. Whether to rename internal package from agentguard to medmesh
2. Whether to extend AgentContext directly
3. Whether to add new GuardVerdict values
4. Whether to introduce OPA/Rego immediately
5. Whether to modify SecurityPipeline tool argument flow
6. Whether to make PHI scrubbing mandatory by default
7. Whether to create healthcare-specific intercept points

---

## Principal Engineer Recommendation

Do not refactor the runtime core yet.

The current runtime contracts are good enough to support the first MedMesh healthcare governance MVP.

The next best step is to analyze the existing guard implementations one by one and identify which guards can be reused, extended, or replaced.

Recommended next analysis document:

    docs/architecture/guard-implementation-analysis.md

