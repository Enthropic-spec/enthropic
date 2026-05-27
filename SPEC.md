# SpeQ Specification v0.3

## 1. Purpose

A `.speq` file is the complete architectural contract of a software project. It is read by an AI agent before generating any code. It defines what exists, what is true, and what is forbidden.

An agent must treat the spec as a closed world. Anything not declared does not exist. Anything declared is binding.

---

## 2. Design Principles

**Closed world.** Entities, layers, transforms, secrets, vocabulary, and classifications not declared in the spec do not exist. An agent must not introduce any element absent from the spec.

**Constraints over instructions.** The spec declares what must be true, not how to achieve it. Unconstrained choices belong to the agent, within declared scope.

**Reproducibility.** Two independent agents reading the same spec must produce architecturally equivalent results. Structural decisions are fixed. Implementation details may vary within constraints.

**Formal and unambiguous.** Purpose-built DSL. No natural language constructs are used as load-bearing syntax. No inference is permitted where a rule exists.

**Immutability.** The spec is read-only for any agent. An agent must not create, edit, or delete a `.speq` file unless the user explicitly requests it in that session. Modifying the spec to reconcile it with generated code is an unconditional security violation.

---

## 3. Primitives

Two primitives underlie the entire format.

**CONTEXT** declares everything that exists: entities, relationships, technology stack, canonical names, organizational layers, and data classifications.

**CONTRACTS** declares everything that must be true regardless of implementation. A contract violation means generated code is rejected, unconditionally.

Four first-class constructs are derived from these primitives:

| Construct  | Primitive          | Purpose                                        |
|------------|--------------------|------------------------------------------------|
| PROJECT    | CONTEXT            | Technology stack and architecture style        |
| VOCABULARY | CONTEXT            | Canonical naming registry                      |
| LAYERS     | CONTEXT+CONTRACTS  | Organizational boundaries and responsibility   |
| FLOWS      | CONTRACTS          | Ordered critical sequences with rollback       |
| TESTING    | CONTRACTS          | Per-flow coverage thresholds and required categories |

These are not optional extensions. They are part of the core grammar.

---

## 4. Format

- **Extension:** `.speq`
- **Encoding:** UTF-8
- **Comments:** `#` to end of line, ignored by parser
- **Keywords:** `UPPERCASE`
- **Identifiers:** `snake_case`
- **Canonical names:** `PascalCase`
- **Layer names:** `UPPER_CASE`
- **Indentation:** two spaces per level (tabs accepted)

---

## 5. Grammar (EBNF)

```ebnf
file                = statement* EOF

statement           = comment
                    | blank_line
                    | version_stmt
                    | project_block
                    | entity_stmt
                    | vocabulary_block
                    | secrets_block
                    | transform_block
                    | layers_block
                    | contracts_block
                    | classify_block
                    | observability_block
                    | testing_block
                    | changelog_block

comment             = "#" <any characters to end of line> NEWLINE
blank_line          = NEWLINE

(* Version *)

version_stmt        = "VERSION" semver NEWLINE
semver              = digit+ "." digit+ "." digit+

(* CONTEXT-derived *)

project_block       = "PROJECT" NEWLINE project_stmt+
project_stmt        = INDENT project_key value NEWLINE
                    | INDENT "DEPS" NEWLINE dep_stmt+
project_key         = "NAME" | "LANG" | "STACK" | "ARCH"
value               = word ("," word)*

dep_stmt            = INDENT INDENT dep_key value NEWLINE
dep_key             = "SYSTEM" | "RUNTIME" | "DEV"

entity_stmt         = "ENTITY" identifier ("," identifier)* NEWLINE

vocabulary_block    = "VOCABULARY" NEWLINE vocab_entry+
vocab_entry         = INDENT PascalName comment? NEWLINE

secrets_block       = "SECRETS" NEWLINE secret_entry+
secret_entry        = INDENT identifier ("->" layer_name)? comment? NEWLINE

transform_block     = "TRANSFORM" NEWLINE transform_rule+
transform_rule      = INDENT identifier "->" identifier ":" identifier ("," identifier)* NEWLINE

layers_block        = "LAYERS" NEWLINE layer_def+
layer_def           = INDENT layer_name NEWLINE layer_stmt+
layer_stmt          = INDENT INDENT layer_kw value NEWLINE
layer_kw            = "OWNS" | "CAN" | "CANNOT" | "CALLS"
                    | "NEVER" | "BOUNDARY" | "EXPOSES" | "LATENCY"

classify_block      = "CLASSIFY" NEWLINE classify_entry+
classify_entry      = INDENT subject classify_class comment? NEWLINE
classify_class      = "credential" | "pii" | "sensitive" | "internal"

(* CONTRACTS-derived *)

contracts_block     = "CONTRACTS" NEWLINE contracts_body+
contracts_body      = contract_rule | flow_block

contract_rule       = INDENT subject constraint comment? NEWLINE
subject             = identifier ("." (identifier | "*"))*
constraint          = "ALWAYS" qualifier
                    | "NEVER" qualifier
                    | "REQUIRES" condition
                    | "AUDIT" audit_field_list
qualifier           = word+
condition           = word (("-" | "_") word)*
audit_field_list    = audit_field ("," audit_field)*
audit_field         = audit_meta | audit_path
audit_meta          = "actor" | "timestamp" | "action" | "outcome"
                    | "target" | "payload-hash" | "signature"
audit_path          = identifier ("." identifier)*

flow_block          = INDENT "FLOW" identifier NEWLINE flow_content+
flow_content        = flow_step | flow_meta
flow_step           = INDENT INDENT digit+ "." layer_tag? subject comment? NEWLINE
layer_tag           = "[" layer_name "]"
flow_meta           = INDENT INDENT flow_key value NEWLINE
flow_key            = "ROLLBACK" | "ATOMIC" | "TIMEOUT" | "RETRY"

(* Observability *)

observability_block = "OBSERVABILITY" NEWLINE obs_entry+
obs_entry           = INDENT "flow" identifier NEWLINE obs_stmt+
obs_stmt            = INDENT INDENT obs_key ":" value NEWLINE
obs_key             = "level" | "must-log" | "must-not-log" | "metrics"

(* Testing *)

testing_block       = "TESTING" NEWLINE test_entry+
test_entry          = INDENT "flow" identifier NEWLINE test_stmt+
test_stmt           = INDENT INDENT test_key ":" test_value NEWLINE
test_key            = "coverage" | "categories" | "performance" | "fixtures"
test_value          = coverage_pct | category_list | perf_list | fixture_list
coverage_pct        = digit+ "%"
category_list       = category ("," category)*
category            = "positive" | "negative" | "boundary"
                    | "security" | "fuzzing" | "performance"
                    | "concurrency" | "rollback" | "idempotency"
perf_list           = perf_assertion ("," perf_assertion)*
perf_assertion      = perf_key ":" word
perf_key            = "p50" | "p95" | "p99" | "throughput" | "concurrency"
fixture_list        = identifier ("," identifier)*

(* Changelog *)

changelog_block     = "CHANGELOG" NEWLINE changelog_version+
changelog_version   = INDENT semver NEWLINE changelog_entry*
changelog_entry     = INDENT INDENT changelog_kw <rest of line> NEWLINE
changelog_kw        = "BREAKING" | "ADDED" | "CHANGED" | "DEPRECATED"

(* Terminals *)

identifier          = lower (lower | digit | "_")*
PascalName          = upper (alpha | digit)*
layer_name          = upper (upper | digit | "_")*
word                = alpha (alpha | digit | "-" | "_")*
digit               = "0".."9"
lower               = "a".."z"
upper               = "A".."Z"
alpha               = lower | upper
INDENT              = "  "
```

---

## 6. Construct Reference

### VERSION

Must be the first non-comment, non-blank statement in every `.speq` file.

```
VERSION 0.2.0
```

---

### PROJECT

Declares the technology context of the project. All choices are binding. An agent must not suggest alternatives or deviate from the declared stack, language, or architecture.

| Key   | Type   | Meaning                                                              |
|-------|--------|----------------------------------------------------------------------|
| NAME  | string | Project name                                                         |
| LANG  | word   | Primary implementation language                                      |
| STACK | list   | Technologies in use, comma-separated                                 |
| ARCH  | word   | Architecture style                                                   |

Common `ARCH` values: `layered`, `event-driven`, `realtime`, `offline-first`, `hexagonal`. Not an exhaustive list.

**DEPS** declares dependencies by installation level. This distinction cannot be expressed by STACK alone.

| Key     | Level                                                              |
|---------|--------------------------------------------------------------------|
| SYSTEM  | OS-level. Must exist before any install step.                      |
| RUNTIME | Managed by the language package manager. Deployed to production.   |
| DEV     | Development only. Must not be present in production.               |

---

### ENTITY

Declares all domain entities. The closed world assumption is absolute: entities not listed here do not exist in this project's model. An agent must not create, reference, or infer entities outside this list.

All identifiers must be `snake_case`.

---

### VOCABULARY

Declares the canonical name for every concept in the project. Each entry is the sole acceptable form across all generated code, file names, variable names, comments, and identifiers. Any deviation is a contract violation.

Entries are `PascalCase`. Comments document what must never be used instead.

---

### SECRETS

Declares the names of all secrets the project requires. This block declares existence only. Values are never written in the spec or any committed file.

`-> LAYER_NAME` scopes a secret to a single layer. An agent must not reference a scoped secret from any other layer. Scoping is a contract, not a convention.

---

### TRANSFORM

Declares all valid directed interactions between entities. Each rule takes the form `source -> target : action(s)`. Only declared transforms are valid. An agent must not generate code for undeclared interactions.

---

### LAYERS

Declares the organizational structure of the system. Layers define ownership, permitted interactions, and absolute prohibitions.

| Key      | Meaning                                                                                  |
|----------|------------------------------------------------------------------------------------------|
| OWNS     | Sole owner of the listed capabilities. No other layer may implement them.                |
| CAN      | Operations this layer is permitted to perform.                                           |
| CANNOT   | Operations explicitly forbidden to this layer.                                           |
| CALLS    | Permitted call targets. Exclusive: calls to any unlisted layer are forbidden.            |
| NEVER    | Absolute prohibition. Equivalent to a CONTRACTS rule scoped to this layer.               |
| BOUNDARY | `external` marks the single entry point for untrusted input. All input must be validated before crossing any layer boundary. At most one layer may declare `BOUNDARY external`. |
| EXPOSES  | Operations accessible from outside the system. Everything not listed is internal.        |
| LATENCY  | Maximum acceptable latency for this layer. Applies to realtime architectures.            |

`CALLS` is exclusive. If a layer declares `CALLS`, calls to any layer not on that list are forbidden.

---

### CLASSIFY

Declares the security classification of entity fields. Classifications are enforced across the entire spec and supersede any conflicting instruction.

| Class      | Agent must                                    | Agent must never                                                    |
|------------|-----------------------------------------------|---------------------------------------------------------------------|
| credential | Encrypt at rest. Never log under any context. | Include in responses, error messages, logs, or traces.              |
| pii        | Handle with data-privacy compliance.          | Log raw. Expose in API responses without explicit declaration.       |
| sensitive  | Restrict to the owning layer.                 | Include in stack traces or debug output.                            |
| internal   | Keep within the owning layer.                 | Expose outside system boundaries.                                   |

A field classified as `credential` is implicitly `must-not-log` in all contexts, regardless of OBSERVABILITY declarations.

---

### CONTRACTS

Declares behavioral invariants. Each rule applies a constraint to a subject. A violated contract means the generated code is unacceptable, unconditionally.

`entity.*` applies the constraint to all operations on that entity.

| Keyword  | Meaning                                                               |
|----------|-----------------------------------------------------------------------|
| ALWAYS   | This condition must hold at all times.                                |
| NEVER    | This state must never occur.                                          |
| REQUIRES | This precondition must be satisfied before the operation proceeds.    |
| AUDIT    | This operation must emit a non-repudiation audit record.              |

---

### AUDIT

`AUDIT` declares a non-repudiation contract: the subject operation must emit an immutable, tamper-evident audit record. Use it for security-critical operations — financial transfers, privilege changes, administrative actions, identity events, configuration changes.

Form: `subject AUDIT field_list`

Each entry in `field_list` is either a reserved meta-field or an entity field. Reserved meta-fields:

| Field          | Meaning                                                          |
|----------------|------------------------------------------------------------------|
| `actor`        | Identity that initiated the operation.                           |
| `timestamp`    | Wall-clock time of the operation.                                |
| `action`       | Operation attempted.                                             |
| `outcome`      | Result of the operation: `success` or `failure`.                 |
| `target`       | Subject of the operation, when distinct from `action`.           |
| `payload-hash` | Cryptographic digest of the request payload.                     |
| `signature`    | Cryptographic signature binding the record to its actor.         |

Semantics:

- An audit record must be emitted for every occurrence of the subject operation, including failures and rollbacks.
- Records must be persisted to an append-only, tamper-evident store. Cryptographic chaining (hash-linked records) or per-record signing is required.
- Records must not be discarded under any condition — timeout, rollback, error, or shutdown. The audit trail is the only authoritative history.
- A field classified `credential` in CLASSIFY must never appear in an AUDIT field list. The validator rejects this.
- AUDIT records are distinct from OBSERVABILITY logs. They have separate stores, separate retention, and separate access controls. An OBSERVABILITY `must-log` declaration does not satisfy an AUDIT contract.

---

### FLOW

Declares an ordered critical sequence. Flows are declared inside a CONTRACTS block and are subject to the same enforcement rules as contract rules.

Steps are numbered from 1, sequential, with no gaps. Minimum 2 steps.

`[LAYER_NAME]` on a step pins that step to the named layer. An agent must implement the step in that layer. If omitted, the agent assigns the step consistent with the LAYERS declaration.

| Key      | Meaning                                                                       |
|----------|-------------------------------------------------------------------------------|
| ROLLBACK | Operations executed on failure, in listed order.                              |
| ATOMIC   | `true`: all steps succeed or the flow fully rolls back. `false`: partial ok.  |
| TIMEOUT  | Maximum wall-clock duration for the complete flow.                            |
| RETRY    | Number of retries permitted on transient failure.                             |

---

### OBSERVABILITY

Declares logging and metrics contracts per flow. These are contracts, not suggestions.

A field classified `credential` in CLASSIFY is `must-not-log` in all flows, whether declared here or not. An agent must not log it regardless of this block.

| Key          | Meaning                                              |
|--------------|------------------------------------------------------|
| level        | Severity: `critical`, `standard`, or `low`.          |
| must-log     | Fields that must always be recorded for this flow.   |
| must-not-log | Fields that must never be recorded for this flow.    |
| metrics      | Metrics that must be emitted for this flow.          |

---

### TESTING

Declares per-flow testing contracts: minimum coverage, required test categories, performance assertions, and required fixtures. Same enforcement as CONTRACTS — a flow that does not satisfy its TESTING contract cannot reach `BUILT` in the state file.

| Key           | Meaning                                                                       |
|---------------|-------------------------------------------------------------------------------|
| `coverage`    | Minimum coverage percentage on the flow's code paths. Form: `N%`.             |
| `categories`  | Required test categories. Every listed category must contribute ≥1 test.      |
| `performance` | Performance assertions verified in load tests (e.g. `p99:200ms, throughput:100rps`). |
| `fixtures`    | Required fixtures that must exist before the flow's tests run.                |

Reserved test categories:

| Category      | Verifies                                                              |
|---------------|-----------------------------------------------------------------------|
| `positive`    | Happy path: declared FLOW steps execute and succeed.                  |
| `negative`    | Error paths: invalid input, rejected requests, expected failures.     |
| `boundary`    | Edge cases: empty input, max sizes, off-by-one, time-zone shifts.     |
| `security`    | Adversarial inputs targeting CONTRACTS: auth bypass, injection, IDOR. |
| `fuzzing`     | Randomized adversarial inputs.                                        |
| `performance` | Latency, throughput, and concurrency limits under load.               |
| `concurrency` | Race conditions, deadlocks, and lock contention.                      |
| `rollback`    | ROLLBACK steps execute correctly when an ATOMIC flow fails mid-sequence. |
| `idempotency` | Retries do not duplicate effects.                                     |

Cross-cutting requirements:

- A FLOW with `ATOMIC true` and a `ROLLBACK` declaration must include `rollback` in its `categories`.
- A FLOW that declares a `RETRY` value must include `idempotency` in its `categories`.
- A FLOW that declares a `TIMEOUT` value must include `performance` in its `categories`, and the load test must assert completion within that timeout.
- A FLOW originating at a layer with `BOUNDARY external` must include `security` in its `categories`.

`performance` assertion keys: `p50`, `p95`, `p99` for latency; `throughput` for requests per unit time; `concurrency` for parallel-request limits.

---

### CHANGELOG

Records the evolution of this spec file. The full changelog is included in AI context blocks so an agent knows what changed between versions and can reason about migrations correctly.

Valid entry keywords:

| Keyword    | Meaning                                                   |
|------------|-----------------------------------------------------------|
| BREAKING   | A change that invalidates previously generated code.      |
| ADDED      | A new construct, entity, layer, or contract.              |
| CHANGED    | A modification to an existing declaration.                |
| DEPRECATED | A construct retained for compatibility but to be removed. |

---

## 7. File System

| File                 | Purpose                                            | Committed |
|----------------------|----------------------------------------------------|-----------|
| `[name].speq`        | The spec. Single source of truth.                  | Yes       |
| `state_[name].speq`  | Build progress per entity, flow, and layer.        | No        |

### state_[name].speq

Generated from the spec. Tracks build progress. Generated automatically by the tool on first `check`.

Status values for CHECKS, TESTS, and AUDIT: `UNVERIFIED`, `OK`, `FAILED`.
Status values for entities, flows, and layers: `PENDING`, `PARTIAL`, `BUILT`.

All `CHECKS` entries must reach `OK` before any entity may be marked `BUILT`. A flow's `TESTS` entries must all be `OK` before the flow may be marked `BUILT`. An entity referenced by an `AUDIT` contract may not be marked `BUILT` until its `AUDIT` entry is `OK`.

```
STATE [name]

  CHECKS
    [lang]              UNVERIFIED
    [dep.system]        UNVERIFIED
    [dep.runtime]       UNVERIFIED

  ENTITY
    [entity]            PENDING

  FLOWS
    [flow]              PENDING

  LAYERS
    [layer]             PENDING

  TESTS
    [flow].coverage     UNVERIFIED
    [flow].[category]   UNVERIFIED
    [flow].performance  UNVERIFIED
    [flow].fixtures     UNVERIFIED

  AUDIT
    [subject]           UNVERIFIED
```

A `TESTS` row is emitted for `coverage`, each declared `categories` entry, `performance` (if declared), and `fixtures` (if declared). An `AUDIT` row is emitted for every `AUDIT` contract subject in the spec.

---

## 8. Validation Rules

A conforming `.speq` file must satisfy all of the following. A file that fails any rule is rejected.

1. `VERSION` is the first non-comment, non-blank statement.
2. `ENTITY` must appear before any block references an entity identifier.
3. Every identifier in `TRANSFORM` is declared in `ENTITY`.
4. Every subject in `CONTRACTS` references a declared entity or uses the `*` wildcard.
5. Every entity referenced in a `FLOW` step is declared in `ENTITY`.
6. `FLOW` step numbers are sequential from 1, with no gaps.
7. Every `FLOW` has at least 2 steps.
8. `LAYERS` names are `UPPER_CASE`.
9. `VOCABULARY` entries are `PascalCase`.
10. `ENTITY` identifiers are `snake_case`.
11. `CALLS` lists may only reference layer names declared in the same `LAYERS` block.
12. `SECRETS` entries are `UPPER_CASE`. No value may appear in the spec.
13. `CALLS` is exclusive. An agent calling a layer not on the `CALLS` list is a contract violation.
14. At most one layer may declare `BOUNDARY external`.
15. A `SECRETS` entry scoped with `->` restricts access to the named layer. Access from any other layer is a contract violation.
16. `CLASSIFY` subjects must match declared entities. The class must be one of `credential`, `pii`, `sensitive`, `internal`.
17. A field classified `credential` must not appear in `must-log` in any `OBSERVABILITY` entry.
18. `FLOW` steps that declare `[LAYER_NAME]` must reference a layer declared in `LAYERS`.
19. Every `AUDIT` contract subject is a declared entity or uses the `*` wildcard.
20. A field classified `credential` in `CLASSIFY` must not appear in any `AUDIT` field list.
21. Every flow referenced in `TESTING` is declared in a `FLOW` block.
22. `TESTING` `coverage` is an integer percentage between 0 and 100, written `N%`.
23. `TESTING` `categories` contains only reserved category identifiers.
24. A `FLOW` with `ATOMIC true` and `ROLLBACK` must include `rollback` in its `TESTING` `categories`.
25. A `FLOW` with a declared `RETRY` value must include `idempotency` in its `TESTING` `categories`.
26. A `FLOW` originating at a layer with `BOUNDARY external` must include `security` in its `TESTING` `categories`.

---

## 9. Scope

SpeQ is domain-agnostic and scale-free. The grammar is identical for a web service, a desktop application, a realtime controller, or a mobile client. Domain vocabulary and entity names are defined per-project. The primitives and grammar do not change.

An agent's semantic understanding of domain terminology comes from training. The spec does not define term semantics. It declares scope, enforces naming, and constrains behavior.

Conforming example specs are in `/examples`.
