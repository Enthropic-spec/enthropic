# Welcome to the SpeQ Wiki

<p align="center">
  <img src="https://raw.githubusercontent.com/speq-ai/speq/main/assets/banner.svg" alt="speq" width="600"/>
</p>

> **SpeQ (Specification for Quality)** is a formal architectural contract format for software development. By writing a `.speq` file once, you ensure that every AI coding session, across any model or platform, operates under a single, unambiguous source of truth. Same spec, any agent, any session: **architecturally identical output**.

---

## 🗺️ Documentation Map

To help you get the most out of SpeQ, we have structured our documentation into three main sections:

### 🎓 Onboarding
*   **[Principal-Level Guide](Principal-Level-Guide)**: A dense, opinionated, architectural guide designed for senior tech leaders. Explains the core theory of mathematical state-space collapse, includes Mermaid system diagrams, and analyzes key tradeoffs.
*   **[Zero-to-Hero Learning Path](Zero-to-Hero-Learning-Path)**: A progressive learning journey for newcomers. Compares natural language specs with SpeQ's closed-world DSL and contains a **40+ term glossary** and key file reference.

### 🚀 Getting Started
*   **[Getting Started with SpeQ](Getting-Started)**: The zero-configuration guide. Learn how to write your first `.speq` file, define domain entities, set up layers, and model transforms.
*   **[How to Load the Skill](How-to-Load-the-Skill)**: A step-by-step workflow for loading `SKILL.md` into AI agents (such as Claude, Gemini, or custom GPTs) to lock in their behavioral contracts.
*   **[State Management & Work Queues](State-Management)**: How to leverage `state_[name].speq` files to coordinate task lists, track progress, and enforce pre-output verification gates.

### 📚 DSL Reference
*   **[EBNF Grammar & Reference](EBNF-Grammar-Reference)**: Complete reference manual of the SpeQ DSL grammar. Detailed descriptions, syntax rules, and practical examples for all context-derived constructs (`PROJECT`, `ENTITY`, `VOCABULARY`, `SECRETS`, `TRANSFORM`, `LAYERS`, `CLASSIFY`) and contract-derived constructs (`CONTRACTS`, `FLOW`, `OBSERVABILITY`, `CHANGELOG`).

---

## 💡 The Core Problem SpeQ Solves

Natural language is inherently ambiguous. The same words are interpreted differently across different models, context lengths, and coding sessions. When AI agents write code using only natural language instructions, they generate high levels of architectural entropy:
*   **Inconsistent Naming:** Naming variables, files, and classes differently every time.
*   **Layer Leaks:** Implementing business logic directly in database access files or UI files.
*   **Security Gaps:** Unintentionally logging sensitive credentials or passing unvalidated inputs deep into core modules.
*   **State Drift:** Straying from original requirements as sessions grow.

**SpeQ collapses this decision space upfront.** By acting as a strict, machine-readable compiler for your software design, SpeQ forces the AI agent to operate inside a closed-world contract. If a model generates code that violates a SpeQ rule, that code is rejected—period.

---

## 🛠️ Contributing & Support

SpeQ is currently in early active development. We welcome contributions to our compiler, CLI tools, and specification format!
*   Refer to the **[Contributing Guide](https://github.com/speq-ai/speq/blob/main/CONTRIBUTING.md)** to get started.
*   Check our **[Security Policy](https://github.com/speq-ai/speq/blob/main/SECURITY.md)** for vulnerability reporting.
*   See the **[Changelog](https://github.com/speq-ai/speq/blob/main/CHANGELOG.md)** for recent specification updates.
