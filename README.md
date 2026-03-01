<p align="center">
  <img src="assets/banner.svg" alt="enthropic"/>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-lightgrey.svg" alt="MIT"/></a>
  &nbsp;
  <a href="SPEC.md"><img src="https://img.shields.io/badge/spec-v0.1.0-ffafff?style=flat&labelColor=0f0f1a" alt="Spec v0.1.0"/></a>
</p>

---

A `.enth` file is the architectural contract of your project — entities, constraints, layer boundaries, naming conventions. You write it once. Every AI session reads it before writing a single line of code.

`.md` files help. Context windows full of documentation do push AI output toward coherence. But natural language is inherently ambiguous — the same words mean different things across models, sessions, and prompts. A structured spec is unambiguous by construction: it has a grammar, a parser, and a validator. The AI cannot misread it.

Same spec, two machines, two agents: architecturally identical output. That's the property.

```
VERSION 1

PROJECT "my-api"
  LANG    python
  STACK   fastapi, postgresql, redis
  ARCH    layered

ENTITY user, session, order

LAYERS
  API     CALLS SERVICE
  SERVICE CALLS STORAGE
  STORAGE

CONTRACTS
  user.password  NEVER    plaintext
  admin.*        REQUIRES verified-auth
```

Not a tool. Not a plugin. A format — like OpenAPI for APIs, like Dockerfile for containers, but for your entire architecture. Readable by humans, unambiguous for AI.

## Why

Vibe coding generates entropy. The AI fills every undeclared decision arbitrarily — naming, layers, security shortcuts — differently every session. Enthropic collapses that space upfront.

The root cause of AI inconsistency isn't the model. It's the missing source of truth.

## Tooling

**[enthropic-tools](https://github.com/Enthropic-spec/enthropic-tools)** — the CLI companion.

```bash
npm install -g enthropic

enthropic new        # create a spec (guided or AI-assisted)
enthropic check      # validate + lint
enthropic context    # generate the context block to feed your AI
enthropic reverse    # reverse-engineer an existing codebase into a starter spec
```

## Files

| File | Purpose | Committed |
|---|---|---|
| `enthropic.enth` | The spec. Source of truth. | ✅ yes |
| `state_[name].enth` | What's built, what's pending. | ❌ no |

## Roadmap

| | |
|---|---|
| v0.2 | Usability: AI-USAGE.md guide + domain examples across common stacks |
| v0.3 | Expressiveness: `DECISIONS` block, richer constraint operators |
| v0.4 | Standard: public spec registry + GitHub Action for CI check |
| v0.5 | Security: stack-aware patterns injected automatically in `context` |

Full spec: [SPEC.md](SPEC.md) · Examples: [examples/](examples/)
