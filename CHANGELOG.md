# Changelog

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
