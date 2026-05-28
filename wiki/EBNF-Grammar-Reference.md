# EBNF Grammar & DSL Reference

This page provides an exhaustive syntax guide for the SpeQ DSL, including EBNF rules and standard definitions for every construct in version 0.3.0.

---

## 📐 The Complete EBNF Grammar

Below is the formal Extended Backus-Naur Form (EBNF) definition for SpeQ, as defined in **[SPEC.md](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/SPEC.md#L62-L188)**:

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

## 🧩 Construct Explanations & Examples

### 1. `PROJECT`
Declares the name of the system, primary programming language, core architecture style, and technical dependencies.
*   **SYSTEM Deps:** Hardware libraries or operating system packages (e.g. `libpq` for PostgreSQL integration).
*   **RUNTIME Deps:** Application libraries installed via standard package managers (e.g. `fastapi`).
*   **DEV Deps:** Build/test dependencies omitted from production artifacts (e.g. `pytest`).

```speq
PROJECT
  NAME   "e_commerce"
  LANG   python
  STACK  fastapi, postgresql
  ARCH   layered
  DEPS
    SYSTEM   libpq
    RUNTIME  fastapi, sqlalchemy
    DEV      pytest, black
```

### 2. `VOCABULARY`
Acts as the canonical naming registry. If a term is declared, it is the **only** acceptable name for that concept across all databases, variable names, functions, files, and annotations.

```speq
VOCABULARY
  AuthToken       # never: jwt, access_token, auth_token, token
  BillingAddress  # never: address, billing, bill_addr
```

### 3. `ENTITY`
A comma-separated list of the only domain objects permitted in the system. Banned entities do not exist and must not be generated.

```speq
ENTITY user, cart, order, payment
```

### 4. `TRANSFORM`
Declares directional actions that entities perform on one another.
```speq
TRANSFORM
  cart  -> order : checkout
  order -> payment : charge
```

### 5. `SECRETS`
Declares the existence of keys and optionally binds them to a target layer.
```speq
SECRETS
  STRIPE_API_KEY -> PAYMENT  # Access is strictly locked to the PAYMENT layer.
```

### 6. `LAYERS`
Enforces architectural boundaries. A layer may own functions exclusively, limit which other layers it calls, or mark itself as the singular `external` input boundary.
*   `BOUNDARY external` - Marks the interface where incoming data must be sanitized before passing into core services.
*   `NEVER` - Prevents the layer from performing specific operations or accessing database handles.

```speq
LAYERS
  FRONTEND
    BOUNDARY  external
    CALLS     BACKEND
    NEVER     calculate_prices
```

### 7. `CONTRACTS`
State invariants that must be validated at compile time or verified during test execution.
*   `ALWAYS` - An absolute condition that must never evaluate to false.
*   `NEVER` - A state that must remain completely unreachable.
*   `REQUIRES` - A prerequisite verification checks before a method executes.
*   `AUDIT` - Specifies which fields or contextual metadata must be logged upon execution of a specific entity action.

```speq
CONTRACTS
  order.total      ALWAYS   positive
  session.token    NEVER    plaintext
  admin.action     REQUIRES authenticated-admin
  payment.charge   AUDIT    actor, timestamp, order.id, payment.status
```

### 8. `FLOW`
Defines an ordered, critical sequence of interactions with transactional metadata.
*   `ATOMIC true` - Enforces standard transaction behavior: either all steps complete, or the whole flow undergoes rollback in sequence.

```speq
  FLOW checkout
    1.  [BACKEND] cart.validate
    2.  [PAYMENT] payment.charge
    3.  [STORAGE] inventory.reserve
    ROLLBACK  payment.void, inventory.release
    ATOMIC    true
    TIMEOUT   30s
```

### 9. `TESTING`
Mandates per-flow test coverage thresholds, categories, performance SLAs, and required fixtures. Code generated for a flow must explicitly contain tests satisfying these parameters.

```speq
TESTING
  flow checkout
    coverage   : 95%
    categories : positive, negative, boundary, rollback, security
    performance: p95:200ms, concurrency:100
    fixtures   : mock_stripe_gateway, mock_inventory_db
```

### 10. `CLASSIFY`
A security block that maps user and system fields to privacy classes: `credential`, `pii`, `sensitive`, or `internal`.
```speq
CLASSIFY
  user.password   credential
  user.email      pii
```
