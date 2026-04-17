# Application Specification Rules

> **Dormant.** This rule file supports the dormant `Abe` persona. For active greenfield and maintained projects, product requirements live in `requirements/` (Pam) and technical specifications live in `tech-specs/` (Tom). These rules apply only when performing a one-time extraction from a project that lacks structured requirements.

## 1. Purpose

Application specifications exist to describe how the application can be recreated or re-implemented from scratch without coupling the specification to the current technology stack.

These specifications are for:

- Future planning agents
- Implementation agents
- Design-review agents
- Rebuild or migration efforts

They are not architecture source code, not UI component docs, and not a copy of the implementation.

## 2. Specification Principles

Application specs must be:

- Technology-neutral
- Architecture-neutral where possible
- Role- and flow-oriented
- Bounded-context aware
- Grounded in current source-of-truth behavior
- Explicit about uncertainty or drift

Application specs must not:

- Prescribe framework-specific implementation details
- Duplicate every current code detail
- Restate generated contracts unnecessarily when they already exist as source of truth
- Infer missing behavior silently

## 3. Source-of-Truth Hierarchy

When building or updating a specification, use this hierarchy:

1. Reviewed product/use-case plans
2. Current domain-model semantics
3. Exported DTOs/OpenAPI/generated types
4. Current implemented behavior in app and service code

If these disagree:

- Prefer the reviewed plan if the implementation is clearly lagging or drifting.
- Prefer the implementation only when it clearly supersedes an older plan.
- Mark unresolved differences as `Needs Review`.

## 4. What Specifications Should Describe

For each feature area, describe:

- Feature purpose
- Primary actors and roles
- Business capabilities
- User/use cases
- Primary user flows
- Domain concepts and their relationships
- API signatures and interaction points
- Page/screen purposes
- Page transitions and page-to-API interactions
- Rules, constraints, and lifecycle behavior
- Known edge cases
- Known deferred areas

## 5. What Specifications Should Not Describe in Detail

Unless a feature genuinely requires it, do not over-specify:

- Component hierarchy
- Styling or layout
- CSS/system design choices
- Framework hooks, state libraries, or transport-layer mechanics
- Internal implementation wiring already implied by exported APIs and types

Screen specs should explain:

- What the page is for
- Who uses it
- What actions it supports
- What backend/API interactions it relies on
- What state transitions matter

Screen specs should not dictate:

- Exact component composition
- Visual spacing
- Grid/flex layouts
- File/module structure for the implementation

## 6. Relationship to API Contracts

Exported API routes and DTO types are the current implementation source of truth.

Therefore:

- Do not duplicate low-level route/request/response details into every screen spec.
- Instead, reference the route group or API surface and summarize its purpose.
- Include API signatures when useful for rebuild guidance:
  - Route
  - Method
  - Purpose
  - Request DTO name
  - Response DTO name
- Avoid copying full generated payload definitions unless the spec is for API reconstruction itself.

## 7. Output Folder Structure

Application specifications should live under `specs/`.

Recommended structure:

```text
specs/
  README.md
  shared/
    glossary.md
    roles-and-actors.md
    navigation-and-entry-points.md
  domains/
    <domain-slug>/
      overview.md
      use-cases.md
      domain-model.md
      api-surface.md
      screens.md
      flows.md
      open-questions.md
```

Where:

- `shared/glossary.md` — canonical terminology; UI term vs model term mappings.
- `shared/roles-and-actors.md` — role definitions and capability summary.
- `shared/navigation-and-entry-points.md` — global navigation, entry points, and cross-domain routing concepts.
- `domains/<domain-slug>/overview.md` — purpose, scope, actors, and current truth summary.
- `domains/<domain-slug>/use-cases.md` — user/use cases and acceptance-style behavioral descriptions.
- `domains/<domain-slug>/domain-model.md` — domain nouns, relationships, lifecycle semantics, and canonical field-level concepts.
- `domains/<domain-slug>/api-surface.md` — route inventory, request/response DTO names, and intent summary.
- `domains/<domain-slug>/screens.md` — page purposes, page actions, data dependencies, and transitions.
- `domains/<domain-slug>/flows.md` — end-to-end flow narratives and state transitions.
- `domains/<domain-slug>/open-questions.md` — drift, ambiguity, deferred work, and review-required items.

## 8. File Content Rules

### `overview.md`

Must include:

- Purpose
- In-scope actors
- Major capabilities
- Dependencies on other domains
- Current implementation status summary

### `use-cases.md`

Must include:

- Actor
- Goal
- Preconditions
- Normal flow
- Postconditions
- Notable edge cases

### `domain-model.md`

Must include:

- Canonical domain concepts
- Relationships
- Lifecycle semantics
- Important invariants
- UI term vs model term mapping when those differ intentionally

### `api-surface.md`

Must include:

- Route/method
- Purpose
- Actor/role allowed
- Request DTO type
- Response DTO type
- Notable behavioral rules

### `screens.md`

Must include:

- Page/screen name
- Route or entry point
- Page purpose
- Key actions
- Key read-only information
- Dependent API calls or query surfaces
- Transitions to other pages or states

### `flows.md`

Must include:

- Trigger
- Main steps
- State transitions
- Alternative/error paths where important

### `open-questions.md`

Must classify each item as one of:

- `Confirmed Drift`
- `Needs Review`
- `Deferred`

## 9. Spec Writing Style

Write specs so a fresh implementation team can use them.

Preferred style:

- Concise prose
- Flat lists where useful
- Tables for route inventories and page inventories
- Explicit labels for confirmed, inferred, and needs-review behavior

Avoid:

- Marketing copy
- Implementation trivia
- Long code excerpts
- Architecture-specific jargon unless the concept is truly part of the product behavior

## 10. Handoff Usefulness Requirement

Every spec should be useful to downstream agents for one or more of:

- Planning a new execution lane
- Reviewing model changes
- Implementing backend contracts
- Implementing frontend screens
- Designing test coverage

If a spec cannot support those downstream tasks, it is too vague and must be improved.

## 11. Change Maintenance

When application behavior changes materially:

- Update the relevant spec files.
- Remove or mark stale behavior explicitly.
- Do not let specs silently drift behind reviewed product truth.

If a spec becomes fully superseded:

- Update the replacement spec in the same slice.
- Mark the old section or file as superseded, or remove it if history is no longer useful.
