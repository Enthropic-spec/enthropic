# Zero-to-Hero Learning Path: Mastering SpeQ

Welcome to SpeQ! Whether you are a software engineer, technical writer, or AI prompt architect, this guide will take you from zero knowledge to a master of SpeQ specifications.

---

## 🗺️ Part I: Foundations & Mental Models

### What is a Behavioral Specification?
Historically, code specifications were static PDF files read by humans. With the advent of AI coding assistants, we have entered the age of **agentic automation**. AI agents do not read specifications to "agree" with them; they execute them as probabilistic constraints. 

If you write a specification in natural language (e.g., "Always ensure passwords are encrypted"), the AI agent translates that instruction through a high-entropy neural pathway. Depending on the temperature, model, and context window, it might implement hashing, salting, plain encryption, or completely ignore it.

**SpeQ is a deterministic compiler for AI behavior.** Instead of asking nicely, you construct a state space where any action other than the specified contract is syntactically invalid.

### Cross-Language Comparisons
To understand the SpeQ DSL, let's look at how SpeQ constraints map to traditional static analysis in other programming languages (like TypeScript):

| Concept | SpeQ Construct | TypeScript Equivalent | Rust Equivalent |
| :--- | :--- | :--- | :--- |
| **Naming constraints** | `VOCABULARY` | Brand Types / Opaque Types | Newtype Pattern |
| **Layer locking** | `LAYERS (OWNS, NEVER)` | ESLint boundaries / Module private imports | Mod privacy boundaries |
| **Preconditions** | `CONTRACTS (REQUIRES)` | Assertions / Type Guards | Custom `TryFrom` traits |
| **Transactional state** | `FLOW (ATOMIC, ROLLBACK)` | Sagas / Try-Catch blocks | Database transactions (`async/await` rolls) |

---

## 📂 Part II: SpeQ Repository Architecture & Domain Model

The SpeQ repository is designed to be extremely lightweight, focusing purely on the specification standards and architectural templates.

### Conceptual Folder Structure
*   📁 **`examples/`**: Contains ready-to-run `.speq` specifications for different architectural scales:
    *   `clock.speq`: Single-file, minimalist Tkinter GUI application.
    *   `chat.speq`: WebSocket-based multi-user real-time chat application.
    *   `shop.speq`: Enterprise-grade, multi-service checkout and payment pipeline.
*   📁 **`docs/`**: Simple, aesthetic web landing page.
*   📄 **`SPEC.md`**: The formal standard definition of the SpeQ grammar.
*   📄 **`SKILL.md`**: The behavioral engine sheet that tells LLMs how to parse, execute, and verify SpeQ constraints.

---

## 🛠️ Part III: Getting Started & Navigating the Repo

### The Three-Step Developer Loop
1.  **Draft the Contract:** Create a `<name>.speq` file in your root folder. Use the EBNF grammar to declare your technology `PROJECT`, canonical terminology `VOCABULARY`, allowed domain `ENTITY` names, layer capability boundaries `LAYERS`, and safety rules `CONTRACTS`.
2.  **Mount the Behavioral Skill:** Pass `SKILL.md` directly into the agent's system prompt or session start context.
3.  **Run the Validation State:** Initialize the work queue by creating `state_main.speq`. The agent will only construct what is marked `PENDING` or `PARTIAL`, updating the state dynamically as tests pass.

---

## 📖 Appendix A: 40-Term SpeQ Glossary

To help you speak the language of structured architecture, master these key terms:

### Primitives & DSL Concepts
1.  **SpeQ**: Specification for Quality. The core framework and contract DSL.
2.  **DSL**: Domain-Specific Language. A mini-programming language tailored to a specific domain (here, architecture specification).
3.  **EBNF**: Extended Backus-Naur Form. The metasyntax notation used to define the SpeQ grammar.
4.  **Closed-World Assumption**: The principle that what is not explicitly declared in the spec does not exist.
5.  **State Space Collapse**: Reducing the massive, probabilistic decision space of an LLM down to a single, structured path.
6.  **Architectural Contract**: A binding agreement that mandates architectural rules and automatically rejects violations.
7.  **Semantic Entropy**: The gradual dilution of meaning and consistency in natural language over time.
8.  **Vibe Coding**: Writing code using natural language instructions without formal constraints, leading to high architectural drift.
9.  **Vibe Drift**: The tendency of codebases to become fragmented and architecturally inconsistent as multiple AI sessions generate code.
10. **Behavioral Contract**: Instructions directed at the AI agent's reasoning engine (such as `SKILL.md`) telling it how to process code.

### Primitives (CONTEXT-derived)
11. **CONTEXT**: A primitive block declaring everything that exists (entities, naming, stack).
12. **PROJECT**: Block containing basic metadata like project name, language, technology stack, and architecture style.
13. **SYSTEM Dependency**: Hardware or OS-level software required to be present before installation (e.g., `libpq`, `tcl-tk`).
14. **RUNTIME Dependency**: Software packages deployed to production environments (e.g., `fastapi`, `sqlalchemy`).
15. **DEV Dependency**: Software packages only used during development or test execution (e.g., `pytest`, `ruff`).
16. **ENTITY**: A core domain object or system component. Anything outside this list is considered an undeclared entity and is banned.
17. **VOCABULARY**: A canonical glossary specifying exactly what names are allowed for concepts, preventing naming variance.
18. **PascalName**: A string formatting rule (`PascalCase`) used for vocabulary canonical registration.
19. **SECRETS**: Block declaring the existence of credential keys, locking them to specific layers.
20. **TRANSFORM**: A declared, directional interaction between two entities (`source -> target : action`).
21. **LAYERS**: High-level logical boundaries separating responsibility in the application code.
22. **BOUNDARY external**: The single, untrusted entry point for external inputs (usually UI or FRONTEND).
23. **OWNS**: Declares the specific software capabilities that a given layer is exclusively allowed to implement.
24. **CALLS**: An exclusive list of layers that a given layer is permitted to invoke.
25. **NEVER (Layer)**: An absolute prohibition preventing a layer from ever executing specific behaviors (e.g., `NEVER compute_time`).
26. **CLASSIFY**: Privacy classification block mapping database fields or entities to security categories.
27. **Credential**: High-security classification. Data classified as a credential must be encrypted at rest and never logged.
28. **PII**: Personally Identifiable Information. Data classified as PII requires strict data privacy handling.
29. **Sensitive**: Data restricted strictly to its owning layer and never included in standard traces.
30. **Internal**: Data kept within the system boundary and never exposed to external clients.

### Primitives (CONTRACTS-derived)
31. **CONTRACTS**: A primitive block enforcing behavioral invariants.
32. **ALWAYS**: A contract clause stating that a specific condition must hold true across all code paths.
33. **NEVER (Contract)**: A contract clause stating that a specific condition must be completely unreachable.
34. **REQUIRES**: A contract precondition that must be validated before an action proceeds.
35. **FLOW**: A numbered, step-by-step critical sequence (like a payment checkout).
36. **ATOMIC**: Flow setting. If `true`, the flow is fully transactional and must rollback entirely upon any step failure.
37. **ROLLBACK**: A list of execution commands triggered in sequence to undo changes when a flow fails.
38. **OBSERVABILITY**: Block dictating what metrics must be gathered and which fields can or cannot be logged.
39. **Must-Log**: Fields that must be present in application logs for a given flow.
40. **Must-Not-Log**: Banned log fields, ensuring sensitive info (like payment tokens) never leaks to standard output.

---

## 📖 Appendix B: Key File Reference

Quick navigation links to the repository's core files:
*   📄 **[README.md](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/README.md)**: Main introduction and project repository status.
*   📄 **[SPEC.md](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/SPEC.md)**: The EBNF grammar specification. Reference this when checking compiler-compliance.
*   📄 **[SKILL.md](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/SKILL.md)**: The AI behavioral prompt blueprint. Load this to enforce strict constraints in coding agents.
*   📁 **[examples/shop.speq](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/examples/shop.speq)**: Standard full-scale reference template.
