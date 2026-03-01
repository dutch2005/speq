# Contributing to the Enthropic Specification

## The Spec Is a Contract

The Enthropic specification defines a format that must be stable and unambiguous.
Changes have downstream effects on every parser, every tool, and every `.enth` file ever written.
Propose carefully.

## Types of Contributions

**Clarifications** — the spec is ambiguous or contradictory. Always welcome.

**New constructs** — must solve a real problem that cannot be expressed with existing primitives.
Open an issue first with a concrete use case and a proposed `.enth` example.

**Grammar fixes** — the EBNF has an error. Open a PR with the fix and a test case.

**Examples** — new `.enth` examples for real domains. Keep them minimal and correct.

## Process

1. Open an issue describing the problem
2. Discuss in the issue — no code first
3. If consensus is reached, open a PR with spec changes + updated examples
4. Grammar changes require an updated EBNF in `SPEC.md`
5. Version bump follows [semver](https://semver.org/): breaking changes → major, new constructs → minor, fixes → patch

## What the Spec Is Not

The spec does not dictate implementation details of tools. It defines the format.
Tools are free to extend beyond the spec as long as they accept all valid `.enth` files.

## Questions

Open a [Discussion](https://github.com/enthropic-spec/enthropic/discussions).
