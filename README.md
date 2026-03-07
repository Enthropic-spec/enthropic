<p align="center">
  <img src="assets/banner.svg" alt="speq"/>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-lightgrey.svg" alt="MIT"/></a>
  &nbsp;
  <a href="SPEC.md"><img src="https://img.shields.io/badge/spec-v0.2-ffafff?style=flat&labelColor=0f0f1a" alt="Spec v0.2"/></a>
</p>

---

A `.speq` file is the architectural contract of your project. You write it once. Every AI session reads it before writing a single line of code. Same spec, two machines, two agents: architecturally identical output.

Natural language is inherently ambiguous. The same words mean different things across models, sessions, and prompts. A `.speq` file has a grammar, a parser, and a validator. It cannot be misread.

<p align="center">
  <a href="SPEC.md"><img src="https://img.shields.io/badge/-Read%20the%20Spec-ffafff?style=for-the-badge&labelColor=0f0f1a" alt="Read the Spec"/></a>
  &nbsp;&nbsp;
  <a href="https://github.com/speq-ai/speq-tools"><img src="https://img.shields.io/badge/-CLI%20Tool-3a3a5c?style=for-the-badge&labelColor=0f0f1a" alt="CLI Tool"/></a>
</p>

## Why

Vibe coding generates entropy. The AI fills every undeclared decision arbitrarily: naming, layers, security, differently every session. SpeQ collapses that space upfront.

AGENTS.md and similar work well, but they are natural language by design, and natural language is not opinionated. On large codebases especially, they drift, get interpreted differently across models and sessions, and cannot be treated as a real source of truth.

The root cause of AI inconsistency is not the model. It is the missing source of truth.

## Examples

Annotated `.speq` files covering different domains and complexity levels: [examples/](examples/)

---

This project is in early development. The format already works, but there is meaningful work left on semantic precision and completeness. Criticism and contributions are very welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) to get involved.
