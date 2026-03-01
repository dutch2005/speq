# Enthropic Specification v0.1.0

## 1. What It Is

Enthropic is a declarative format specification for AI-assisted development.

A `.enth` file is the complete architectural map of a project. It is read by an AI agent
before generating any code. It eliminates entropy by making all architectural decisions
explicit, permanent, and machine-readable.

A `.enth` file is not code. It does not generate code by itself. It collapses the space
of acceptable code an AI can generate — leaving only the region consistent with the
decisions already made.

---

## 2. Design Principles

**Closed world assumption.**
Anything not declared in the spec does not exist for the AI agent. The boundary is
implicit in the scope of the declaration, not a separate construct.

**Constraints over instructions.**
The spec does not tell the AI *how* to build. It defines what must be true regardless
of how. Choices that are not constrained are left to the AI's judgment within the
declared scope.

**Reproducibility.**
Given the same `.enth` file, two independent AI agents — in different sessions, on
different machines — must produce architecturally equivalent results. Structural
decisions are identical. Implementation details may vary within the constraints.

**Human-readable, machine-parseable.**
Purpose-built DSL. Uppercase keywords, lowercase identifiers, minimal punctuation.
No indentation ambiguity. Unambiguous at a glance.

---

## 3. Two Primitives

Everything in Enthropic is built on exactly two primitives.

### CONTEXT

The closed world. Declares everything that exists in the project: entities, directed
relationships between them, the technology stack, canonical names, and organizational
layers. What is not declared in CONTEXT does not exist in the project's conceptual model.

### CONTRACTS

The invariants. Declares everything that must be true regardless of implementation:
behavioral constraints, sequential requirements, and responsibility boundaries. A contract
violation means the generated code is unacceptable — no exceptions, no overrides.

**Four first-class derived constructs** extend these primitives with readable syntax for
patterns that appear in every non-trivial project:

| Construct | Primitive | What it adds |
|---|---|---|
| `PROJECT` | CONTEXT | Technology meta-context (stack, arch, language) |
| `VOCABULARY` | CONTEXT | Canonical naming registry for this project |
| `LAYERS` | CONTEXT + CONTRACTS | Organizational boundaries and responsibility rules |
| `FLOWS` | CONTRACTS | Ordered critical sequences with rollback semantics |

These are not optional extensions. They are part of the core grammar.

---

## 4. Format Foundation

- **File extension:** `.enth`
- **Encoding:** UTF-8
- **Comments:** `#` to end of line, ignored by parser
- **Keywords:** `UPPERCASE`
- **Entity identifiers:** `snake_case`
- **Vocabulary (canonical names):** `PascalCase`
- **Layer names:** `UPPER_CASE`
- **Indentation:** two spaces per level (tabs accepted)

---

## 5. Formal Grammar (EBNF)

```ebnf
file            = statement* EOF

statement       = comment
                | blank_line
                | version_stmt
                | project_block
                | vocabulary_block
                | secrets_block
                | entity_stmt
                | transform_block
                | layers_block
                | contracts_block

comment         = "#" <any characters to end of line> NEWLINE
blank_line      = NEWLINE

(* ── Version ───────────────────────────────────────── *)

version_stmt    = "VERSION" semver NEWLINE
semver          = digit+ "." digit+ "." digit+

(* ── CONTEXT-derived constructs ────────────────────── *)

project_block   = "PROJECT" NEWLINE project_stmt+
project_stmt    = INDENT project_key value NEWLINE
                | INDENT "DEPS" NEWLINE dep_stmt+
project_key     = "NAME" | "STACK" | "ARCH" | "LANG"
value           = word ("," word)*

dep_stmt        = INDENT INDENT dep_key value NEWLINE
dep_key         = "SYSTEM" | "RUNTIME" | "DEV"

secrets_block   = "SECRETS" NEWLINE secret_entry+
secret_entry    = INDENT identifier comment? NEWLINE

vocabulary_block = "VOCABULARY" NEWLINE vocab_entry+
vocab_entry     = INDENT PascalName comment? NEWLINE

entity_stmt     = "ENTITY" name_list NEWLINE
name_list       = identifier ("," identifier)*

transform_block = "TRANSFORM" NEWLINE transform_rule+
transform_rule  = INDENT identifier "->" identifier ":" action_list NEWLINE
action_list     = identifier ("," identifier)*

layers_block    = "LAYERS" NEWLINE layer_def+
layer_def       = INDENT layer_name NEWLINE layer_stmt+
layer_stmt      = INDENT INDENT layer_kw value NEWLINE
layer_kw        = "OWNS" | "CAN" | "CANNOT" | "CALLS" | "NEVER" | "LATENCY"

(* ── CONTRACTS-derived constructs ──────────────────── *)

contracts_block = "CONTRACTS" NEWLINE contracts_body+
contracts_body  = contract_rule | flow_block

contract_rule   = INDENT subject constraint comment? NEWLINE
subject         = identifier ("." (identifier | "*"))*
constraint      = always_c | never_c | requires_c
always_c        = "ALWAYS" qualifier
never_c         = "NEVER" qualifier
requires_c      = "REQUIRES" condition
qualifier       = word+
condition       = word (("-" | "_") word)*

flow_block      = INDENT "FLOW" identifier NEWLINE flow_content+
flow_content    = flow_step | flow_meta
flow_step       = INDENT INDENT digit+ "." subject "." identifier comment? NEWLINE
flow_meta       = INDENT INDENT flow_key value NEWLINE
flow_key        = "ROLLBACK" | "ATOMIC" | "TIMEOUT" | "RETRY"

(* ── Terminals ─────────────────────────────────────── *)

identifier      = lower (lower | digit | "_")*
PascalName      = upper (alpha | digit)*
layer_name      = upper (upper | digit | "_")*
word            = alpha (alpha | digit | "-" | "_")*
semver          = digit+ "." digit+ "." digit+
digit           = "0".."9"
lower           = "a".."z"
upper           = "A".."Z"
alpha           = lower | upper
INDENT          = "  "   (* two spaces *)
```

---

## 6. Construct Reference

### VERSION

Must be the first non-comment statement in the file.

```
VERSION 0.1.0
```

---

### PROJECT *(CONTEXT-derived)*

Declares the technology context. Stack choices made here are binding. The AI does not
suggest alternatives or second-guess these decisions.

| Key | Type | Meaning |
|---|---|---|
| `NAME` | string | Human-readable project name |
| `STACK` | list | Technologies in use, comma-separated |
| `ARCH` | word | Architecture style (`layered`, `event-driven`, `realtime`, `offline-first`, etc.) |
| `LANG` | word | Primary implementation language |

#### `DEPS` sub-block *(optional)*

Declares dependencies by installation level. Distinguishes what must exist at the OS
level from what the package manager handles — a distinction `STACK` cannot express.

| Key | Level | Examples |
|---|---|---|
| `SYSTEM` | OS-level. Must exist before any install step. Not managed by the language package manager. | `tcl-tk`, `libpq`, `openssl`, `rust-toolchain` |
| `RUNTIME` | Language package manager. Deployed to production. | `fastapi`, `stripe`, `react` |
| `DEV` | Development only. Never in production. | `pytest`, `ruff`, `typescript` |

```
PROJECT
  NAME   "WorldClock"
  STACK  python, tkinter, zoneinfo
  ARCH   layered
  LANG   python

  DEPS
    SYSTEM   tcl-tk
    RUNTIME  zoneinfo
    DEV      pytest
```

```
PROJECT
  NAME   "CNC Controller"
  STACK  c, linux_kernel, posix_realtime
  ARCH   realtime
  LANG   c
```

---

### VOCABULARY *(CONTEXT-derived)*

Declares the canonical names for this project. Every entry is the sole acceptable form
for that concept across all generated code, comments, file names, and identifiers.
Naming drift is a contract violation.

Entries are `PascalCase`. Comments document what must never be used instead.

```
VOCABULARY
  AuthToken       # never: auth_token, JwtToken, accessToken, jwt
  UserId          # never: user_id, uid, uuid
  OrderStatus     # never: order_state, status, order_status
```

---

### ENTITY *(CONTEXT)*

Declares all domain entities. The closed world assumption applies: entities not listed
here do not exist in this project's model.

All entity names must be `snake_case` and must be declared before being referenced
in `TRANSFORM`, `LAYERS`, or `CONTRACTS`.

```
ENTITY user, product, cart, order, payment, session, admin
```

---

### TRANSFORM *(CONTEXT)*

Declares directed relationships between entities. Each rule defines one valid
interaction path: `source -> target : action(s)`.

Multiple actions on the same transform are comma-separated. Only declared transforms
are valid. Undeclared interactions do not exist.

```
TRANSFORM
  user  -> cart    : add_product, remove_product
  cart  -> order   : checkout
  admin -> product : create, update, delete
```

---

### SECRETS *(CONTEXT)*

Declares the key names of all secrets this project requires. Declaring a name here
is the complete obligation — values are never written anywhere in the spec.

**Why this block exists:**
- The AI always knows *what* secrets the project needs, from the spec
- Values never pass through chat, never appear in version control, never in logs
- The vault status file (`vault_[name].enth`) mirrors this list with `SET` / `UNSET` per key
- Actual values live encrypted in `~/.enthropic/[project].secrets` — never in the repo

```
SECRETS
  DATABASE_URL     # postgres connection string
  STRIPE_KEY       # stripe secret key
  JWT_SECRET       # signing key for AuthToken
```

---

### LAYERS *(CONTEXT + CONTRACTS derived)*

Declares the organizational layers of the system, their ownership, permitted
interactions, and absolute prohibitions.

| Key | Meaning |
|---|---|
| `OWNS` | Sole owner of these capabilities. No other layer may implement them. |
| `CAN` | Permitted operations for this layer. |
| `CANNOT` | Explicitly forbidden. Stronger than not listing: actively prohibited. |
| `CALLS` | Permitted dependencies. Unlisted layers may not be called. |
| `NEVER` | Absolute prohibition, contract-level. Equivalent to a CONTRACTS rule. |
| `LATENCY` | For realtime layers: maximum acceptable latency. |

```
LAYERS
  BACKEND
    OWNS    business_logic, auth, payment_orchestration
    CALLS   DATABASE, STRIPE
    NEVER   trust_client_calculated_total
    NEVER   store_plaintext_credentials
```

---

### CONTRACTS *(CONTRACTS)*

Declares behavioral invariants. Each rule applies a constraint to a subject.

Wildcards: `entity.*` applies to all operations involving that entity.

| Keyword | Meaning |
|---|---|
| `ALWAYS` | This condition must hold at all times. |
| `NEVER` | This state must never occur. |
| `REQUIRES` | This precondition must be satisfied before the operation proceeds. |

```
CONTRACTS
  checkout.total       ALWAYS   server-side
  payment.credentials  NEVER    backend-stored
  admin.*              REQUIRES verified-admin-role
```

---

### FLOW *(CONTRACTS-derived)*

Declares an ordered critical sequence. Steps are numbered, must execute in the
declared order, and are part of the CONTRACTS block.

| Meta-key | Meaning |
|---|---|
| `ROLLBACK` | Comma-separated operations to execute on failure, in listed order. |
| `ATOMIC` | `true`: entire flow must succeed or fully roll back. `false`: partial completion is acceptable. |
| `TIMEOUT` | Maximum wall-clock duration for the complete flow. |
| `RETRY` | Retry policy on transient failure. |

```
CONTRACTS
  FLOW checkout
    1. cart.validate
    2. order.create
    3. payment.authorize
    4. inventory.reserve
    5. payment.capture
    6. order.confirm
    7. inventory.decrement
    8. notification.send
    ROLLBACK  payment.void, inventory.release, order.cancel
    ATOMIC    true
    TIMEOUT   30s
```

---

## 7. Project File System

| File | Purpose | Committed |
|---|---|---|
| `enthropic.enth` | The spec. Single source of truth. Declares secret key names (never values). | Yes |
| `state_[name].enth` | Runtime state. What is built, what is pending. Includes CHECKS for DEPS. | No |
| `vault_[name].enth` | Secret key status mirror (`SET` / `UNSET`). Human-readable. No values. | Never |
| `~/.enthropic/[name].key` | Fernet encryption key. Never leaves the machine. chmod 600. | Never |
| `~/.enthropic/[name].secrets` | Encrypted JSON blob with actual secret values. Never in repo. | Never |

All non-committed files are auto-created by `enthropic validate`.

### state_[name].enth

Tracks build progress and environment check status. Generated from the spec.

```
STATE project_name

  CHECKS
    python                       UNVERIFIED   # LANG
    tcl-tk                       UNVERIFIED   # DEPS.SYSTEM
    requests                     UNVERIFIED   # DEPS.RUNTIME

  ENTITY
    user                         PENDING
    session                      PENDING

  FLOWS
    login                        PENDING

  LAYERS
    BACKEND                      PENDING
    DATABASE                     PENDING
```

All CHECKS must reach `OK` before any ENTITY can be marked `BUILT`.

### vault_[name].enth

Tracks secret key status. Never contains values. Generated and refreshed from spec.

```
VAULT project_name

  DATABASE_URL                 UNSET
  STRIPE_KEY                   SET
  JWT_SECRET                   UNSET
```

Values are stored encrypted in `~/.enthropic/[name].secrets` (Fernet AES-128-CBC + HMAC-SHA256).
The encryption key lives in `~/.enthropic/[name].key` (chmod 600).

Workflow:

```bash
enthropic vault set DATABASE_URL "postgres://..."    # encrypts + updates status to SET
enthropic vault keys                                 # lists key names only, never values
enthropic vault export --out .env                    # explicit decrypt to .env (deployment only)
```

---

## 8. Validation Rules

A conforming `.enth` file must satisfy all of the following:

1. `VERSION` is the first non-comment, non-blank statement.
2. `ENTITY` must be declared before `TRANSFORM`, `LAYERS`, or `CONTRACTS` reference any entity.
3. Every entity referenced in `TRANSFORM` is declared in `ENTITY`.
4. Every subject in `CONTRACTS` references a declared entity or uses the `*` wildcard.
5. Every entity in a `FLOW` step is declared in `ENTITY`.
6. `FLOW` step numbers are sequential, starting from 1, with no gaps.
7. Every `FLOW` has at least 2 steps.
8. `LAYERS` names are `UPPER_CASE`.
9. `VOCABULARY` entries are `PascalCase`.
10. `ENTITY` identifiers are `snake_case`.
11. `VAULT` blocks must not appear in `enthropic.enth`.
12. A `LAYERS` block that declares `CALLS` may only list layer names declared within the same `LAYERS` block.
13. `SECRETS` entries must be `UPPER_CASE` identifiers. Values must not appear in the spec.

---

## 9. Scope and Domains

Enthropic is **domain-agnostic** and **scale-free**.

The grammar is identical for an e-commerce platform, a CNC machine controller, a
mobile notes app, a 3D renderer, or an OS kernel. The entities and vocabulary change.
The primitives do not. Granularity is determined by project scope — a spec can describe
a full system or a single module.

The AI agent's semantic understanding of domain terminology (what `spindle`, `gcode`,
`jwt`, `inventory` mean) comes from training. The spec's job is not to define semantics
— it is to declare scope, enforce naming, and constrain behavior.
