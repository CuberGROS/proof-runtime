# Contributing to Proof Runtime

Thank you for your interest in Proof Runtime. The project is **pre-alpha**
and in **Phase 0**: repository foundation and architecture documentation.

## What is in scope right now

- Review of the architecture documents for clarity, consistency and
  correctness.
- Discussion of the [open questions](docs/adr/0001-architecture-freeze.md#open-questions).
- Identification of contradictions between documents, or between the
  documents and the [invariants](docs/invariants.md).
- Proposals for new Architecture Decision Records (ADRs).

## What is out of scope right now

- Runtime or protocol implementation code.
- Protocol field definitions or schemas, until a specification phase is
  authorized.
- New dependencies, frameworks, plugins or tooling unrelated to the
  documentation.
- Claims about performance, security or production readiness that are not
  backed by evidence.

## Changing the architecture

The four planes, the four core protocols and the invariants are frozen by
[ADR 0001](docs/adr/0001-architecture-freeze.md). To change any of them:

1. Open an issue describing the problem and the proposed change.
2. Submit a new ADR in `docs/adr/` using the next free number
   (`NNNN-short-title.md`). ADRs are never renumbered or deleted; a
   superseded ADR is marked as such and links to its replacement.
3. Update every affected document in the same pull request.

A change that introduces a **new architectural primitive** (a new plane
component, a new protocol, or a new invariant) must say so explicitly in the
pull request description so that reviewers can evaluate it as such.

## Documentation conventions

- All repository documentation is written in **English**.
- Prefer precise, restrained wording. State the boundary in which any
  guarantee holds.
- Do not invent protocol fields or behavior. Record unresolved decisions as
  open questions.
- Diagrams use Mermaid so they render on GitHub and stay reviewable as text.
- Follow [.editorconfig](.editorconfig) for whitespace and line endings.

## Pull requests

- Work on a branch; do not push directly to `main`.
- Keep pull requests focused on one topic.
- Describe what changed and why, and list any open questions the change
  raises or resolves.
- Do not commit secrets, credentials, generated build artifacts or local
  environment files.

## Licensing of contributions

This project is licensed under the [Apache License, Version 2.0](LICENSE).
Unless you explicitly state otherwise, any contribution you intentionally
submit for inclusion is licensed under the same terms, as described in
Section 5 of the license.

## Security issues

Do not report security issues in public. Follow [SECURITY.md](SECURITY.md).
