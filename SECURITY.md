# Security Policy

## Scope

The SpeQ specification itself (`.speq` format) does not handle secrets, keys, or sensitive data.
The spec only declares *names* of secrets — never values.

Security concerns about **tooling** (CLI, parsers) should be reported to:
[https://github.com/speq-ai/speq-tools/security/advisories/new](https://github.com/speq-ai/speq-tools/security/advisories/new)

## Spec-Level Security Concerns

If you identify a security issue in the **specification design** itself — for example, a construct that could encourage insecure patterns — open a private advisory or email the maintainers.

Do not open a public issue for security concerns until a fix is available.
