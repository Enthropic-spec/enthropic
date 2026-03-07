# Contributing

The SpeQ specification is in early development. The format works, but semantic precision and completeness still need serious work. Criticism is as valuable as code — both are welcome.

## What to contribute

**Ambiguities** — something in the spec can be read two ways, or a rule does not cover an edge case. Always welcome.

**Semantic gaps** — a real architectural concern cannot be expressed with the current constructs. Open an issue with a concrete case and a proposed syntax.

**Grammar corrections** — the EBNF is wrong or incomplete. Open a PR with the fix.

**Examples** — a new `.speq` example for a domain not yet covered. Keep it minimal and correct.

**Validation rules** — a case that should be a hard error but is not caught.

## Process

1. Open an issue describing the problem
2. Discuss before writing any spec changes
3. If there is consensus, open a PR with the change and updated examples
4. Breaking grammar changes require an updated EBNF in `SPEC.md`
5. Versioning follows [semver](https://semver.org/): breaking changes → major, new constructs → minor, corrections → patch

## What the spec is not

The spec defines a format. It does not dictate how tools implement it. Tools may extend behavior as long as they accept all valid `.speq` files.

## Questions

Open a [Discussion](https://github.com/speq-ai/speq/discussions).
