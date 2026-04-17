# Technical Specification Rules

These rules govern the technical specification artifacts that `Tom` (Technical Specification Creator) produces, with domain-model work owned by `Dom` (Data Modeler). All technical specifications live under `tech-specs/` and serve as the canonical technical reference for Archie, Brad, and Fran.

For the Tom persona playbook (operating sequence, JIT invocation, team orchestration), see `agents/technical-specification-creator.md`.

---

## 1. Scope

Technical specifications describe **how the product works at the feature level** — domain model, API surface, and data flows that translate product intent into implementable structure. They do not describe:

- Product requirements (Pam's territory, in `requirements/`)
- Cross-cutting architecture decisions (Archie's territory, in `plans/`)
- Implementation details (framework patterns, file structure, component hierarchy)
- CI/CD, deployment, or infrastructure

---

## 2. Required Artifacts

### Per Feature (`tech-specs/features/<feature-slug>/`)

| File | Owner | Contents |
|---|---|---|
| `domain-model.md` | Dom (reviewed by Tom) | Fields table, relationships, state machines, invariants per entity |
| `api-surface.md` | Tom | Route inventory with method, route, purpose, DTOs, roles, errors |
| `flows.md` | Tom | Per use case: trigger, sequence, error branches, state transitions |
| `open-questions.md` | Tom | Technical ambiguities classified as `Confirmed Drift`, `Needs Review`, `Deferred` |

---

## 3. Domain Model (`domain-model.md`)

Dom produces this file and enforces `rules/domain-model-conventions-rules.md`.

### Per Entity

**Fields table** (required):

| Name | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `id` | UUID | No | generated | PK |

**Relationships** (required when the entity has associations):

- Cardinality (one-to-one, one-to-many, many-to-many).
- Cascade semantics (what happens to related records on delete/inactivate).
- Ownership direction.

**State machine** (required for any lifecycle field — `status`, `isActive`, etc.):

- States and allowed transitions.
- Who/what triggers each transition.
- Mermaid `stateDiagram-v2` preferred; transition table acceptable.

**Invariants** (required when the entity has cross-field constraints):

- Uniqueness constraints beyond single-field unique indexes.
- Cross-entity integrity rules.
- Ordering or sequencing guarantees.

### Convention Enforcement

- Follow `rules/domain-model-conventions-rules.md` for lifecycle naming (`isActive` vs `status`), soft-delete vs hard-delete, DTO alignment, enum-backed modeling, and filtering conventions.
- If a proposed model departs from conventions, justify the departure explicitly in the domain-model file.

---

## 4. API Surface (`api-surface.md`)

### Route Inventory Table (Required)

| Method | Route | Purpose | Request DTO | Response DTO | Allowed Roles | Notable Errors |
|---|---|---|---|---|---|---|
| `GET` | `/api/v1/...` | ... | none | `FooResponse` | member, commissioner | `NOT_FOUND` |

### Rules

- Every route must trace back to at least one use case in Pam's `use-cases.md`.
- Every route must declare allowed roles explicitly — no implicit "anyone authenticated."
- Every route must list notable domain errors (not just HTTP status codes, but domain-specific error codes like `LEAGUE_NOT_FOUND`, `ALREADY_MEMBER`).
- Request and response DTO names must be listed; full field definitions belong in `domain-model.md`, not repeated here.
- Add behavioral notes below the table when route semantics aren't obvious (pagination style, sort behavior, idempotency, rate-limit posture).

---

## 5. Flows (`flows.md`)

### Per Use Case Block (Required)

For each use case from Pam's `use-cases.md`:

- **Trigger** — what initiates the flow (user action, system event, scheduled job).
- **Sequence** — screen → API → service → persistence interaction. Name the route from `api-surface.md` and the entity from `domain-model.md`.
- **Error branches** — what happens on validation failure, auth denial, not-found, conflict. Reference the notable errors from `api-surface.md`.
- **State transitions** worth noting — which entity field changes, from what to what.

### Rules

- Do not duplicate Pam's use-case prose. Reference `requirements/features/<feature>/use-cases.md` by use-case ID; describe the technical *how*, not the product *what*.
- Every use case must have a corresponding flow. If a use case has no technical flow (pure UI state, no API call), state that explicitly.
- Error branches must reference domain error codes from `api-surface.md`, not generic HTTP statuses.

---

## 6. Open Questions (`open-questions.md`)

Same classification scheme as Pam's requirements:

| Classification | Meaning |
|---|---|
| `Confirmed Drift` | Implementation and spec disagree; the divergence is known |
| `Needs Review` | A technical decision is needed before implementation can proceed |
| `Deferred` | Acknowledged gap, explicitly out of scope for this feature |

### Rules

- Every `Needs Review` item must state the specific question or decision needed.
- `Deferred` items must include rationale for deferral.
- `open-questions.md` must be empty (or contain only `Deferred` items) before handoff to downstream agents.

---

## 7. Cross-Check Requirements

Before marking the tech spec complete, Tom must verify:

- [ ] Every Pam use case has a corresponding block in `flows.md`.
- [ ] Every screen action in Pam's `screens.md` implies at least one route in `api-surface.md`.
- [ ] Every route traces back to at least one use case.
- [ ] Every route references entities in `domain-model.md`.
- [ ] Naming matches Pam's `glossary.md` — no drift between product terminology and technical terminology.

---

## 8. Handoff Floor — Ready for Downstream

Technical specifications are not ready to hand off until:

- [ ] All per-feature files exist: `domain-model.md`, `api-surface.md`, `flows.md`, `open-questions.md`.
- [ ] Every entity has a fields table and (if it has lifecycle) a state machine.
- [ ] Every route has an allowed-role declaration and notable errors.
- [ ] Every Pam use case has a corresponding flow.
- [ ] `open-questions.md` is empty or contains only `Deferred` items with explicit rationale.
- [ ] Naming matches Pam's glossary.
- [ ] PSE or owner has reviewed the tech spec.

---

## 9. JIT Invocation

Tom can be triggered automatically when an implementation request arrives for a feature area:

- If `requirements/features/<feature>/` exists but `tech-specs/features/<feature>/` does not, Tom runs before any code is written.
- If `tech-specs/features/<feature>/` exists but is incomplete (some files missing or stale), Tom completes the missing pieces.
- The human does not need to explicitly start a tech-spec phase — the system recognizes the gap and fills it.

---

## 10. What Technical Specifications Must Not Do

- Invent product behavior to fill gaps. If a use case is missing, route it back to Pam.
- Duplicate Pam's content. Reference `requirements/` files; don't copy use-case text into `flows.md`.
- Prescribe implementation details. Describe *what* the system does technically, not *how* it's coded.
- Add extensibility points or configuration layers for hypothetical future requirements.
- Depart from `rules/domain-model-conventions-rules.md` without explicit justification.
