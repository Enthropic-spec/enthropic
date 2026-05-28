# How to Load the SpeQ Skill

The `.speq` file defines *what* is true in your architecture. The `SKILL.md` file defines *how* the AI coding assistant must behave when working with your spec. Together, they create a robust, self-correcting development cycle.

This guide explains how to properly load, reference, and enforce `SKILL.md` across various AI coding agents.

---

## 🛠️ Step 1: Provide the Skill to the Agent

Always load the `SKILL.md` alongside your `.speq` contract at the **absolute start** of your AI session.

### Method A: Native Agent IDE Extensions (Gemini, Claude Code, Cursor)
If you are using a command-line agent or IDE assistant that supports file references, pass both files in your first prompt:

```bash
# Example command in Claude Code or Antigravity
I want to start a new feature. Read SKILL.md and project.speq first.
```

### Method B: Chat Interfaces (ChatGPT, Claude.ai Web UI)
If using a web-based chat assistant:
1.  Upload `SKILL.md` and your `.speq` file as attachments.
2.  Paste this initialization prompt:
    > "I have uploaded two files: `SKILL.md` (your behavioral contract) and `project.speq` (the architectural contract). Read both completely. Do not write any code until you confirm you have internalized all entries in the `VOCABULARY`, `CONTRACTS`, and `CLASSIFY` blocks."

---

## 🚦 Step 2: Run a Pre-Flight Verification

Before letting the agent generate any implementation files, run a quick diagnostic check to ensure it has properly parsed the spec constraints:

```markdown
"Before writing any code, list:
1. Every domain ENTITY that is allowed to exist.
2. Every VOCABULARY term and its banned aliases.
3. Every CONTRACTS rule we must satisfy.
4. Any classified credential fields and their logging restrictions."
```

If the agent correctly lists these items based *only* on the spec, it is ready. If it hallucinates or suggests an entity not present in your `.speq` file, immediately prompt: **"Violation of Closed-World Assumption. Re-read SKILL.md and adjust."**

---

## 🛑 Step 3: Enforcing Absolute Red Lines

The SpeQ Skill mandates strict enforcement of **12 Absolute Red Lines** (found in **[SKILL.md](file:///C:/Users/maertzdorf/OneDrive/GitHub/speq/SKILL.md#L151-L167)**). During code generation, if the AI violates any of these, reject the output and point them to the exact rule:

*   **Banned Names:** If the AI writes `user_id` instead of `UserId` (a `VOCABULARY` violation), tell the agent:
    > "Contract violation: Used unauthorized variable name `user_id` instead of canonical `UserId` defined in VOCABULARY."
*   **Layer Leaks:** If a database query is written inside a file labeled as `GATEWAY`, tell the agent:
    > "Contract violation: The GATEWAY layer is calling DB directly. This violates the allowed CALLS pathway: GATEWAY -> SERVICE -> DB."

Using these enforcement methods, you keep your generated codebase aligned with your original architecture.
