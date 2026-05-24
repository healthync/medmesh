# External Guardrails Analysis: HealthClaw Guardrails

## Objective

Analyze the public HealthClaw Guardrails GitHub repository and determine:

- how it overlaps with MedMesh
- how it differs from MedMesh
- what MedMesh can learn from it
- how MedMesh should avoid becoming a clone
- how MedMesh should position itself clearly

Source reviewed:

    https://github.com/aks129/HealthClawGuardrails

Important note:

This analysis is based on the publicly visible GitHub README and repository structure, not a full source-code audit.

---

## HealthClaw Guardrails Summary

HealthClaw Guardrails positions itself as:

    The security layer between AI agents and clinical data.

Its README states that it is a vendor-neutral guardrail proxy that sits between any AI agent and any FHIR server.

The stated request flow is:

    AI Agent -> MCP Server -> Guardrail Proxy -> Any FHIR Server

Core features described in the README include:

- PHI redaction
- immutable audit trail
- step-up authorization
- human-in-the-loop writes
- tenant isolation
- medical disclaimers
- compiled truth timelines
- FHIR R4 and R6 support
- MCP tools
- upstream FHIR proxying
- wearable data mapping into FHIR Observations
- FHIR Permission evaluation
- provenance tracking
- data quality evaluation through Curatr

---

## HealthClaw Technical Focus

HealthClaw appears to focus heavily on:

1. FHIR server proxying
2. MCP tool access to FHIR
3. FHIR read/write guardrails
4. PHI redaction on FHIR reads
5. tenant isolation
6. step-up authorization for writes
7. human-in-the-loop approval for clinical writes
8. immutable audit trail
9. FHIR R6 Permission evaluation
10. patient-owned data quality workflows

This makes it a concrete healthcare interoperability guardrail project.

---

## Areas of Overlap With MedMesh

HealthClaw overlaps with MedMesh in several important areas.

### 1. Healthcare AI Guardrails

Both projects care about making healthcare AI safer.

Shared concerns:

- safe agent access to clinical data
- PHI protection
- auditability
- authorization
- human review
- tenant isolation

---

### 2. PHI Governance

HealthClaw includes PHI redaction on reads and a de-identification endpoint.

MedMesh also plans PHI governance as a core capability.

Overlap:

- PHI detection
- PHI redaction
- de-identification
- masking before AI exposure
- output/input safety

---

### 3. FHIR Access Governance

HealthClaw is strongly FHIR-oriented.

MedMesh also plans FHIR-aware authorization.

Overlap:

- FHIR resources
- FHIR operations
- FHIR validation
- read/write control
- SMART/FHIR context
- resource-level guardrails

---

### 4. Auditability

HealthClaw includes immutable audit trails and AuditEvent export.

MedMesh also plans auditability as a core governance feature.

Overlap:

- audit logs
- decision tracing
- clinical data access history
- provenance tracking

---

### 5. Human-in-the-Loop

HealthClaw requires human confirmation for clinical writes.

MedMesh also plans human review gates.

Overlap:

- review required for high-risk operations
- write approval
- clinical safety escalation

---

## Important Differences

Despite the overlap, MedMesh should not be positioned as the same thing.

The key differences are architectural scope and abstraction level.

---

## Difference 1: Proxy Product vs Runtime Mesh

HealthClaw appears to be primarily a FHIR guardrail proxy.

MedMesh should be a governance runtime mesh.

### HealthClaw

Primary pattern:

    agent -> MCP -> FHIR proxy -> FHIR server

### MedMesh

Target pattern:

    application / agent framework / MCP / FHIR / model / tool -> governance mesh -> approved execution

MedMesh should govern:

- prompts
- model calls
- tool calls
- FHIR calls
- non-FHIR tools
- agent runtime behavior
- PHI transformations
- policy decisions
- observability
- deployment profiles

This makes MedMesh broader than a FHIR proxy.

---

## Difference 2: FHIR-Centric vs Agent-Runtime-Centric

HealthClaw is very FHIR-centric.

MedMesh should be agent-runtime-centric.

HealthClaw asks:

    How do we protect FHIR access from AI agents?

MedMesh should ask:

    How do we govern healthcare AI agents across their full execution lifecycle?

MedMesh lifecycle stages include:

- user input
- prompt construction
- pre-LLM inspection
- model routing
- tool authorization
- pre-tool validation
- tool output inspection
- final output inspection
- async audit
- telemetry
- policy evaluation
- human review

This is a broader runtime model.

---

## Difference 3: Concrete FHIR App vs Framework/SDK

HealthClaw appears to ship a working Flask application, FHIR proxy, MCP server, dashboard, routes, demos, and workflows.

MedMesh should become a framework and platform layer.

MedMesh should provide:

- Python SDK
- guard pipeline
- policy runtime
- healthcare guard interfaces
- FHIR adapter interfaces
- observability hooks
- pipeline profiles
- deployment patterns
- reusable governance primitives

Rather than only being one proxy app, MedMesh should be embeddable into many healthcare AI systems.

---

## Difference 4: MCP-Focused vs Multi-Framework

HealthClaw is strongly tied to MCP tool access and Claude plugin workflows.

MedMesh should support MCP eventually, but should not be limited to MCP.

MedMesh should support:

- LangGraph
- LangChain
- direct Python agents
- REST gateway mode
- MCP mode
- custom healthcare services
- future agent frameworks

This gives MedMesh a wider developer platform scope.

---

## Difference 5: FHIR Read/Write Guardrails vs Full AI Governance

HealthClaw includes FHIR read/write protection.

MedMesh should include full AI governance primitives.

MedMesh should govern:

- PHI leakage
- prompt injection
- jailbreak attempts
- unsafe clinical output
- model routing
- tool authorization
- patient context boundaries
- consent context
- audit events
- policy decisions
- data minimization
- deployment risk profiles

The difference is that MedMesh should govern the whole AI execution path, not only the FHIR access path.

---

## Difference 6: Product Surface

HealthClaw product surface:

- FHIR REST endpoints
- MCP tools
- dashboard
- plugin
- proxy mode
- demo workflows

MedMesh product surface should be:

- governance SDK
- AI gateway
- policy runtime
- PHI gateway
- FHIR adapter
- audit engine
- observability layer
- runtime profiles
- agent framework adapters

---

## What MedMesh Can Learn From HealthClaw

HealthClaw is useful as a reference.

MedMesh can learn from:

### 1. FHIR Proxy Pattern

HealthClaw demonstrates that an intermediary layer between agents and FHIR servers is valuable.

MedMesh can adopt the idea but generalize it into a broader governance mesh.

---

### 2. PHI Redaction On Reads

This is important.

MedMesh should inspect tool outputs before data enters model context.

This validates our planned TOOL_OUTPUT PHI guard.

---

### 3. Step-Up Authorization For Writes

This is a strong pattern.

MedMesh should support step-up controls for high-risk actions such as:

- creating FHIR resources
- updating medications
- changing allergies
- deleting resources
- exporting patient data
- sending patient messages

---

### 4. Human-in-the-Loop Writes

This is essential in healthcare.

MedMesh should support clinical review gates for:

- write operations
- patient-facing recommendations
- medication changes
- care plan updates
- high-risk outputs

---

### 5. Tenant Isolation

MedMesh must support tenant-aware governance.

This is especially important for:

- hospitals
- clinics
- payers
- SaaS deployments
- multi-tenant healthcare AI platforms

---

### 6. Provenance and Compiled Truth

HealthClaw’s compiled truth idea is useful.

MedMesh can learn from this but frame it more broadly as:

    evidence-aware AI execution lineage

Instead of only showing current resource state, MedMesh can track:

- prompt lineage
- model call lineage
- tool call lineage
- FHIR resource lineage
- policy decision lineage
- output lineage

---

### 7. FHIR R6 Permission Evaluation

FHIR Permission is relevant for future healthcare authorization.

MedMesh should track this as a possible integration point, but should not depend entirely on FHIR R6 because it is still experimental/ballot-stage.

---

## What MedMesh Should Avoid Copying

MedMesh should avoid copying HealthClaw’s exact product shape.

Avoid becoming:

- just a Flask FHIR proxy
- just an MCP plugin
- just a redaction endpoint
- just a FHIR demo app
- just a patient data quality tool

MedMesh should remain a reusable healthcare AI governance platform.

---

## MedMesh Differentiated Positioning

Recommended positioning:

    MedMesh is the governance mesh for healthcare AI agents.

Expanded positioning:

    MedMesh provides runtime governance, PHI safety, policy enforcement, FHIR-aware authorization, and auditability across healthcare AI agent workflows.

MedMesh should not position itself as:

    a FHIR proxy only

Instead, it should position itself as:

    runtime infrastructure for governed healthcare AI systems

---

## Differentiation Table

| Dimension | HealthClaw Guardrails | MedMesh |
|---|---|---|
| Primary focus | FHIR/MCP guardrail proxy | Healthcare AI governance runtime |
| Main boundary | AI agent to FHIR server | Agent, model, tool, data, policy, audit |
| Core architecture | Flask proxy plus MCP tools | Guard pipeline, SDK, gateway, policy runtime |
| FHIR support | Central product feature | One governed adapter layer |
| PHI support | Redaction and de-identification endpoints | Runtime PHI governance across input, tools, output |
| Tooling | MCP and Claude plugin focused | Multi-framework: LangGraph, LangChain, MCP, REST |
| Governance model | Proxy-level controls | Runtime governance across execution lifecycle |
| Audit | Immutable FHIR/AuditEvent trail | Cross-agent execution lineage and governance telemetry |
| HITL | Clinical writes | Writes, risky outputs, policy escalations |
| Product type | Working healthcare proxy app | Platform/framework for healthcare AI agents |
| Research angle | Practical FHIR guardrails | Generalizable zero-trust governance architecture for healthcare AI agents |

---

## Strategic Conclusion

HealthClaw is not a reason to stop MedMesh.

It is evidence that the problem is real.

However, it also means MedMesh must be clearly differentiated.

The correct MedMesh path is:

1. Do not build only a FHIR proxy.
2. Do not build only MCP tools.
3. Do not build only PHI redaction.
4. Build a broader runtime governance layer.
5. Use FHIR as one adapter inside a larger governance mesh.
6. Make policy, PHI, audit, observability, and agent runtime first-class.
7. Support multiple agent frameworks and deployment modes.

---

## Impact On MedMesh Roadmap

This analysis changes the roadmap slightly.

MedMesh should prioritize:

1. Runtime pipeline profiles
2. Governance metadata contracts
3. PHI gateway
4. FHIR authorization guard
5. Audit engine
6. Agent framework adapters
7. MCP support later

MCP should not be the first defining feature.

FHIR proxy should not be the whole product.

The first defining feature should be:

    governed healthcare AI agent runtime

---

## Next Step

Continue with:

    docs/architecture/medmesh-pipeline-profiles.md

This will define how MedMesh runtime profiles differ across:

- development
- internal clinical assistant
- external model strict mode
- research de-identification
- patient-facing assistant
- FHIR tool governance

