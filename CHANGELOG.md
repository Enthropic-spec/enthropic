# Changelog

## [0.3.0]

- Added `AUDIT` constraint keyword in `CONTRACTS`: non-repudiation contracts for security-critical operations. Declares the field list that must appear in every emitted audit record. Reserved meta-fields: `actor`, `timestamp`, `action`, `outcome`, `target`, `payload-hash`, `signature`.
- Added `TESTING` block: per-flow `coverage` thresholds, required `categories`, `performance` assertions, and required `fixtures`. Reserved categories: `positive`, `negative`, `boundary`, `security`, `fuzzing`, `performance`, `concurrency`, `rollback`, `idempotency`.
- Audit records must be persisted to an append-only, tamper-evident store. They are distinct from `OBSERVABILITY` logs and have separate retention.
- A field classified `credential` cannot appear in any `AUDIT` field list (cross-validated with `CLASSIFY`).
- Cross-cutting `TESTING` requirements: `ATOMIC true` + `ROLLBACK` flows require `rollback` category; `RETRY` flows require `idempotency`; `TIMEOUT` flows require `performance`; flows originating at `BOUNDARY external` require `security`.
- Added validation rules 19-26.
- Extended state file: `TESTS` and `AUDIT` sections track per-flow categories and per-subject audit verification.
- Removed `Roadmap` section: deferred items now implemented.

## [0.2.0]

- Added `CLASSIFY` block: security classification for entity fields (`credential`, `pii`, `sensitive`, `internal`).
- Added `OBSERVABILITY` block: logging and metrics contracts per flow.
- Added `CHANGELOG` block: records spec evolution, included in AI context.
- Added `BOUNDARY external` keyword in LAYERS: declares the single entry point for untrusted input.
- Added `EXPOSES` keyword in LAYERS: declares the public surface of the system.
- Added `->` scoping syntax in SECRETS: restricts a secret to a named layer.
- Added `[LAYER]` prefix on FLOW steps: pins a step to a specific layer.
- Added Immutability principle: an agent must not modify a `.speq` file without explicit user request.
- Added validation rules 13-18.
- Removed OWNERSHIP block: human governance, outside spec scope.
- Removed QUOTAS block: outside spec scope.
- Removed PERFORMANCE block: outside spec scope.
- Updated examples: clock, chat, shop.
- Removed examples: cnc, notes.

## [0.1.0]

First release.

- Two primitives: CONTEXT and CONTRACTS.
- Four first-class constructs: PROJECT, VOCABULARY, LAYERS, FLOWS.
- Formal EBNF grammar.
- 13 validation rules.
- Examples: shop, cnc, notes.
