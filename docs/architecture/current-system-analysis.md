# Current System Analysis

## Source Project
Original codebase: `agentguard`

## Product Direction
MedMesh will evolve this project from a general AI agent guardrail framework into:

> The governance mesh for healthcare AI agents.

## Current Strengths

### 1. Runtime Foundation
The project already has a strong composition root through `AgentFramework`.

Current setup order:
1. Auth
2. Logging
3. Observability
4. Security
5. Adapter instrumentation

This is useful for MedMesh because healthcare AI systems need controlled runtime execution.

---

### 2. Security Pipeline
The current security system supports:

- input checks
- output checks
- tool authorization checks
- prompt injection detection
- jailbreak detection
- budget enforcement
- tool scope enforcement
- async monitoring

This can become the foundation for:

- PHI inspection
- clinical prompt governance
- healthcare policy checks
- unsafe clinical action blocking

---

### 3. Auth Layer
The existing auth layer includes:

- JWT verification
- JWKS support
- Keycloak integration
- OPA policy evaluation
- token delegation

This is highly relevant to:

- SMART on FHIR
- OAuth2/OIDC
- clinician identity
- patient-scoped authorization
- least-privilege agent execution

---

### 4. Observability Layer
The existing observability layer includes:

- OpenTelemetry tracing
- span propagation
- Langfuse integration
- security event recording

This can become the foundation for:

- clinical AI audit trails
- agent execution lineage
- PHI leakage monitoring
- policy decision tracing
- healthcare compliance dashboards

---

### 5. Plugin Architecture
The plugin system is important because healthcare integrations are highly extensible.

Future MedMesh plugins may include:

- FHIR adapters
- RxNorm tools
- ICD-10 tools
- LOINC tools
- PHI detectors
- clinical policy guards
- audit sinks
- EHR connectors

---

### 6. Server Mode
The project already exposes an HTTP API for:

- input checks
- output checks
- tool checks
- batch checks
- streaming checks
- health checks

This is important because MedMesh should support non-Python agents and enterprise service deployment.

---

## Current Gaps

### Healthcare Domain Gaps

The current system does not yet include:

- FHIR resource awareness
- PHI-specific healthcare governance
- HIPAA Safe Harbor workflows
- SMART on FHIR integration
- clinical workflow policies
- healthcare audit semantics
- patient-scoped access models
- terminology service integrations

---

## Strategic Decision

For now, we will NOT immediately rename the internal Python package from `agentguard` to `medmesh`.

Reason:

- avoid breaking working code
- preserve test stability
- first understand architecture
- refactor gradually once MedMesh boundaries are clear

MedMesh is the product/platform identity.

`agentguard` remains the temporary internal engine name.

---

## Initial Component Mapping

| Current Component | Future MedMesh Role |
|---|---|
| `security/` | PHI Safety + Runtime Guardrails |
| `auth/` | Identity, SMART on FHIR, Policy Enforcement |
| `observability/` | Auditability + Clinical AI Tracing |
| `logging/` | Compliance Logging |
| `plugins/` | Healthcare Extension Framework |
| `adapters/` | AI Framework + Model Provider Abstraction |
| `server/` | MedMesh Gateway API |
| `prompts/` | Prompt Governance Layer |
| `docker/`, `deploy/` | Enterprise Deployment Foundation |

---

## Next Analysis Areas

1. Security pipeline deep dive
2. Auth pipeline deep dive
3. Observability pipeline deep dive
4. Plugin architecture review
5. Server/API review
6. Healthcare gap analysis
7. MedMesh target architecture
8. MVP extraction plan

