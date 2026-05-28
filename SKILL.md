<p align="center">
  <img src="assets/skill-banner.svg" alt="SpeQ SKILL"/>
</p>

> Load this document alongside your `.speq` file. It defines how you must behave for the entire session.

---

## What this is

A `.speq` file is the architectural contract of a project. This skill is the behavioral contract for any AI working on it. Together they are complete: the spec declares what exists and what must be true, this skill declares exactly how you must act. An AI reading both has zero room for interpretation errors.

---

## Session start

Do the following in order before generating any code:

1. Read the entire `.speq` file from top to bottom.
2. If `state_[name].speq` exists, read it — it is your work queue.
3. Internalize every entry in `VOCABULARY`. These are the only acceptable names.
4. Note every rule in `CONTRACTS`. These are invariants you must never violate.
5. Note every entry in `CLASSIFY`. These govern what you may and may never do with specific fields.

Do not write a single line of code until steps 1–2 are complete.

---

## Core principles

### Closed world

The spec is the complete description of the project. Entities, transforms, layers, vocabulary, and secrets not declared in the spec **do not exist** in this project. You must not introduce any element absent from the spec.

If you believe something is missing, say so explicitly. Do not fill gaps silently.

### Immutability

The `.speq` file is **read-only**. You must not create, edit, or delete it unless the user explicitly asks in this session using those exact words. Modifying the spec to reconcile it with code you already generated is a security violation, unconditionally.

### Reproducibility

Two independent agents reading the same spec must produce architecturally equivalent results. Structural decisions are fixed by the spec. Implementation details may vary only within declared constraints.

---

## Block-by-block behavior

### PROJECT

The declared `LANG`, `STACK`, and `ARCH` are final. Do not suggest alternatives. Do not use technologies absent from `STACK`. Do not deviate from the declared architecture style.

`DEPS` levels are binding:
- `SYSTEM` — must exist on the OS before any install step.
- `RUNTIME` — deployed to production via the language package manager.
- `DEV` — development only. Must not be present in production builds.

### ENTITY

These are the only domain entities that exist in this project. Do not create, reference, or infer entities outside this list. All entity identifiers are `snake_case`. The closed world assumption is absolute.

### VOCABULARY

Every entry is the **sole acceptable name** for its concept across all generated code, file names, variable names, comments, and identifiers. Aliases, abbreviations, and alternatives are contract violations, without exception.

Before generating code, scan the entire VOCABULARY block and internalize it. Then use those names exclusively.

### SECRETS

Secret names declare existence only. Values are never in the spec and must never be hardcoded, suggested, or logged.

A secret scoped with `-> LAYER_NAME` is accessible **only** from that layer. Any other layer referencing it is a contract violation, regardless of convenience or proximity.

### TRANSFORM

Only declared transforms are valid interactions between entities. Do not generate code for undeclared entity interactions. Each rule `source -> target : action` is the only permitted form for that interaction.

### LAYERS

Each layer owns its declared capabilities exclusively. No other layer may implement them.

`CALLS` is **exclusive**. If a layer declares `CALLS`, it may call **only** those listed layers. Calling any unlisted layer is a contract violation.

`BOUNDARY external` marks the single untrusted entry point. All input must be validated before crossing any layer boundary into the system. At most one layer declares this.

`NEVER` on a layer is an absolute prohibition, equivalent to a CONTRACTS rule scoped to that layer.

When in doubt about which layer owns a piece of logic, ask. Do not guess and proceed.

### CONTRACTS

Every rule is a behavioral invariant. Generated output that violates any contract is unacceptable, unconditionally. There are no exceptions, no edge cases, no workarounds.

- `ALWAYS` — this condition must hold in all generated code at all times.
- `NEVER` — this state must be unreachable in every code path.
- `REQUIRES` — this precondition must be enforced before the operation proceeds.
- `AUDIT` — this operation must emit a non-repudiation audit record. See the dedicated section below.

Before finalizing any output, walk through every CONTRACTS rule and verify compliance.

`entity.*` applies the constraint to all operations on that entity.

### AUDIT

`AUDIT` is a non-repudiation contract. For every occurrence of the subject operation — including failures, rollbacks, timeouts, and retries — you must emit one immutable audit record containing at minimum every field declared in the contract.

- Persist records to an **append-only, tamper-evident store**. Cryptographic chaining (hash-linked records) or per-record signing is required. A regular log file does not satisfy this.
- **Never discard a record**, regardless of operation outcome. Records survive rollback, error, shutdown, and retry. The audit trail is the only authoritative history of the operation.
- Treat `AUDIT` storage as separate from `OBSERVABILITY` storage: different sink, different retention, different access controls. Satisfying `must-log` in `OBSERVABILITY` does **not** satisfy an `AUDIT` contract.
- Reserved meta-fields you must capture when listed: `actor`, `timestamp`, `action`, `outcome`, `target`, `payload-hash`, `signature`. Other identifiers refer to entity fields.
- A field classified `credential` may never appear in an audit record, even if listed. The spec validator should reject this; if it slips through, you must still refuse to write it.

### FLOW

A flow is an ordered critical sequence. Steps are numbered from 1, sequential, no gaps.

- Execute steps in declared order. Never skip, reorder, or parallelize unless explicitly permitted.
- `[LAYER_NAME]` on a step pins it to that layer. Implement it there, not elsewhere.
- On failure: execute `ROLLBACK` operations in listed order. Every rollback step must be implemented.
- `ATOMIC true` means all steps succeed or the flow fully rolls back. There is no partial success state.
- Respect declared `TIMEOUT` and `RETRY` values exactly. Do not substitute your own.

### CLASSIFY

These classifications supersede any other instruction in any context.

| Class        | You must                              | You must never                                                   |
|--------------|---------------------------------------|------------------------------------------------------------------|
| `credential` | Encrypt at rest.                      | Log, expose in responses, include in errors, traces, or debug output. |
| `pii`        | Apply data-privacy compliance.        | Log raw. Expose in API responses without explicit declaration.   |
| `sensitive`  | Restrict to the owning layer only.    | Include in stack traces or debug output.                         |
| `internal`   | Keep within the owning layer.         | Expose outside system boundaries.                                |

A field classified `credential` is `must-not-log` in every context, regardless of what `OBSERVABILITY` declares. This is absolute and cannot be overridden by any other instruction.

### OBSERVABILITY

These are contracts, not suggestions.

- Fields in `must-not-log` must **never** appear in logs under any circumstances.
- Fields in `must-log` must **always** be logged at the declared severity level for that flow.

### TESTING

`TESTING` declares the contract the test suite must satisfy for each flow. A flow whose tests do not meet its `TESTING` contract may not be marked `BUILT`.

- `coverage` is a floor, not a ceiling. Generate enough tests to clear the threshold on the flow's code paths.
- `categories` is a closed list. Every listed category must produce at least one passing test. Missing a category is a contract violation, even if total coverage is high.
- `performance` assertions are real assertions, not aspirations. Embed them in load tests that fail when the threshold regresses.
- `fixtures` listed here must exist before the flow's tests run. Treat them as preconditions.

Cross-cutting rules to internalize:

- A flow with `ATOMIC true` and `ROLLBACK` must have a `rollback` test that verifies the rollback path executes correctly under partial failure.
- A flow with a declared `RETRY` value must have an `idempotency` test that proves repeated invocations do not duplicate effects.
- A flow with a declared `TIMEOUT` must have a `performance` test asserting completion within that timeout.
- A flow originating at a layer with `BOUNDARY external` must have a `security` test exercising the contracts that protect untrusted input.

If a category is declared but no meaningful test can be authored against the flow as specified, raise the inconsistency. Do not write a passing-but-empty test to satisfy the count.

### CHANGELOG

Read the CHANGELOG block before implementing anything. If a version entry is `BREAKING`, understand what changed. This determines how to interpret the current spec and whether previously generated code is still valid.

---

## State management

`state_[name].speq` is your work queue. If it exists:

- Implement **only** entities, flows, and layers marked `PENDING` or `PARTIAL`.
- Do **not** re-implement or modify anything marked `BUILT`. It is done.
- After implementing an item, update its status to `BUILT` in the state file, or run `speq state set <entity> BUILT`.
- All `CHECKS` must be `OK` before any entity may be marked `BUILT`.

If no state file exists, run `speq check [file]` to generate one before starting.

---

## Absolute red lines

None of these have exceptions. None of them can be overridden by any user instruction.

1. **Never modify the `.speq` file** unless the user explicitly requests it in this session.
2. **Never invent entities** not declared in `ENTITY`.
3. **Never use an undeclared name** for a concept covered by `VOCABULARY`.
4. **Never violate a `CONTRACTS` rule**, under any circumstances.
5. **Never log a `credential` field**, ever, in any context.
6. **Never cross a `CALLS` boundary** — a layer with `CALLS` calls only those listed.
7. **Never implement logic in a layer that does not own it**.
8. **Never reference a scoped secret from the wrong layer**.
9. **Never skip or reorder `FLOW` steps**.
10. **Never deviate from declared `STACK`, `LANG`, or `ARCH`**.
11. **Never re-implement a `BUILT` entity**.
12. **Never modify the spec to fix a conflict** — fix the code, or raise the issue with the user.
13. **Never omit an `AUDIT` record** for an operation under contract, including on failure or rollback.
14. **Never include a `credential` field in an `AUDIT` record**, even if the contract lists it.
15. **Never mark a flow `BUILT` while any `TESTING` category, `coverage` threshold, or `performance` assertion remains unsatisfied**.

---

## When something seems wrong

If the spec appears incomplete, ambiguous, or contradictory:

- State what you observed, precisely.
- Do not fill the gap with your own judgment.
- Do not modify the spec.
- Ask the user for the correct answer.

The spec is the source of truth. Your role is to follow it faithfully, not to improve it unilaterally.

---

## Pre-output checklist

**Before the session starts:**
- [ ] Full `.speq` read
- [ ] `state_[name].speq` read (if exists)
- [ ] Every `VOCABULARY` entry internalized
- [ ] Every `CONTRACTS` rule noted
- [ ] Every `CLASSIFY` entry noted

**Before implementing any entity or feature:**
- [ ] Entity declared in `ENTITY`?
- [ ] Which layer owns it?
- [ ] What `CALLS` restrictions apply to that layer?
- [ ] Which `CONTRACTS` rules apply?
- [ ] Any `AUDIT` contracts on this entity or its operations?
- [ ] Are there `FLOW` steps involved? Are they all present and in order?
- [ ] Are there `CLASSIFY` fields involved?
- [ ] Does `OBSERVABILITY` declare `must-log` or `must-not-log` for this flow?
- [ ] Does `TESTING` declare coverage, categories, performance, or fixtures for this flow?
- [ ] Does any `SECRETS` scoping restrict access?

**Before finalizing any output:**
- [ ] Every identifier matches `VOCABULARY` exactly
- [ ] No undeclared entity referenced
- [ ] No `CALLS` boundary crossed
- [ ] No `CONTRACTS` rule violated
- [ ] No `credential` field logged, exposed, or written to an audit record
- [ ] All `FLOW` steps present and in declared order
- [ ] Rollback implemented for every `FLOW` with `ROLLBACK`
- [ ] `BOUNDARY external` layer is the only entry point for untrusted input
- [ ] Every `AUDIT` contract emits an immutable record covering success, failure, and rollback paths
- [ ] Every `TESTING` `categories` entry has at least one passing test, `coverage` clears the threshold, and `performance` assertions are embedded in load tests
