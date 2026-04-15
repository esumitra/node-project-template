# Domain Model Conventions Rules

These rules define consistency conventions for the domain model itself. Use them when proposing, reviewing, or implementing schema/entity/DTO shape changes.

This file is intentionally separate from [model-change-rules.md](model-change-rules.md):

- `domain-model-conventions-rules.md` defines **what a consistent model should look like**.
- `model-change-rules.md` defines **the process/checklist for changing a model safely**.

---

## 1. Purpose

The domain model should use consistent patterns so that:

- Data-modeler recommendations are repeatable.
- Backend and frontend interpret entity lifecycle the same way.
- DTOs remain aligned with domain semantics rather than drifting into ad hoc per-feature patterns.
- Future model changes can build on established conventions instead of re-deciding basics every time.

---

## 2. Lifecycle Naming Conventions

### Active vs Inactive

Default convention for soft-delete-style lifecycle:

- Use `isActive: boolean`.
- `true` means the record is active in normal product flows.
- `false` means the record remains persisted but should normally be filtered out of active/default views or treated as read-only/inactive.

Use `isActive` by default for:

- Soft delete
- Inactive records that remain queryable
- Read-only preserved records
- Eligibility gating before a later hard delete

When an entity uses `isActive`, it must be a first-class persisted field in the storage model itself.

- Do not model `isActive` only in DTOs.
- Do not hide `isActive` inside JSON/settings blobs when it is a true lifecycle concept for the entity.
- Expose the same first-class `isActive` concept consistently through shared domain types and relevant DTOs.

Do not invent multiple lifecycle shapes for the same meaning across entities without a documented reason.

### Hard Delete

Permanent delete should usually mean actual row removal.

- Hard delete removes the record.
- Do not add `deletedAt` just to accompany a true hard delete.
- Only add tombstone-style fields such as `deletedAt` when the product explicitly needs retained deleted-record metadata rather than real removal.

### Status

Reserve `status` for workflow or business-state progression, not basic soft-delete semantics.

Examples of appropriate `status` usage:

- Workflow lifecycle (draft → submitted → processed)
- Invitation lifecycle
- Approval flow
- Processing progress

Examples of what should usually **not** use `status`:

- Simple active/inactive lifecycle
- "Soft deleted" records that only need normal filtering

If both concepts are needed, keep both:

- `status` for workflow/business meaning
- `isActive` for record activity/presence

Do not collapse them together for convenience.

### Inactive Reasons

If future business needs require distinguishing *why* something is inactive, prefer a separate field such as `inactiveReason`.

This keeps the primary active/inactive filter simple while still supporting richer lifecycle semantics later. Do not over-model reason enums preemptively when current product behavior only needs active vs inactive.

---

## 3. DTO Conventions

DTOs should preserve the same lifecycle semantics as the domain model.

- If an entity uses `isActive`, its API-facing DTOs should normally expose `isActive` rather than translating that concept into a different lifecycle field name.
- Do not use DTO-only `status` values to represent soft delete when the domain model uses `isActive`.
- If a DTO intentionally differs from the domain model, document the boundary reason explicitly in the active plan or code comments.

Lifecycle wording should stay consistent across:

- Prisma schema
- Shared domain types
- DTOs
- Service logic
- Route documentation
- Frontend UI copy where practical

When a field is intentionally constrained to a closed set of values, model it as an enum/union rather than a broad string.

Use enum-backed modeling for:

- Persistence schema enums when the storage tier supports them and the value set is stable
- Shared domain enums (e.g. `packages/shared/domain/enums.ts`)
- Shared domain types
- DTO schemas
- Mapper return types

Do not use free-form `string` when the product meaning is actually:

- A known provider set
- A known format choice
- A known policy choice
- A known workflow state

If persistence still stores a broad string temporarily, document that as transitional debt and keep the API/domain surface strongly typed.

For closed sets that are stable enough to persist strongly:

- Use a first-class enum in the persistence schema rather than a broad `String` column.
- Keep mapper logic explicit when persistence enum identifiers differ from the API/domain literals.
- Normalize legacy values during migration rather than preserving stale aliases indefinitely.

Do not leave persistence as free-form text once the repo has high confidence that the values are closed, reviewed, and actively used in product/API flows.

---

## 4. Filtering Conventions

Default "active" product views should usually filter on `isActive=true` when the entity uses this convention.

When inactive records remain user-visible:

- Document that explicitly in the contract and UI plan.
- Make the difference between visible-but-inactive and active clear in route docs and DTO field descriptions.

Do not rely on tribal knowledge for whether inactive rows should still appear.

---

## 5. Future-State Guidance

When designing new entities, start with the simplest lifecycle model that matches the approved product behavior.

Preferred progression:

1. No lifecycle field when the entity is always ephemeral or always hard-deleted.
2. `isActive` when the entity needs a simple soft-delete / inactive state.
3. Add `inactiveReason` later if product meaning genuinely requires it.
4. Add `status` only when workflow/business-state tracking is required.

Do not jump straight to enums or multi-state workflow models without real product need.

---

## 6. Data-Modeler Responsibilities

When reviewing a proposed model change, the data-modeler should explicitly check:

- Whether the lifecycle concept is actually soft delete, workflow state, or both.
- Whether `isActive` is the correct primary field.
- Whether `status` is being misused to stand in for simple active/inactive semantics.
- Whether DTOs will remain semantically aligned with the proposed model.
- Whether a proposed extra field is truly needed now or should be deferred.

If the proposal breaks these conventions, the data-modeler should call that out before backend implementation begins.

---

## 7. Evolving These Conventions

Update this file when the repo-wide domain conventions themselves change, not merely when a specific feature is being implemented. New conventions should be promoted only after reviewer confirmation, and preferably only when the pattern is visible in multiple domain areas or clearly beneficial as a forward-looking default.
