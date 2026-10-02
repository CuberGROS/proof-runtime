# Security Policy

## Project status

Proof Runtime is **pre-alpha**. The repository currently contains
architecture documentation only. There is no released software, and no
version of this project is supported for production use.

Nothing in this repository should be read as a claim that any implementation
is secure, hardened or audited. Security properties described in the
architecture documents are **design intentions** that apply only within the
enforcement boundary the runtime actually controls (see
[docs/architecture/overview.md](docs/architecture/overview.md#integration-grades)).

## Supported versions

| Version | Supported |
| ------- | --------- |
| none    | n/a       |

## Reporting a vulnerability

Please do **not** disclose security issues in public issues, pull requests or
discussions.

1. Use GitHub's **private vulnerability reporting** for this repository
   (the "Report a vulnerability" button under the Security tab), if it is
   enabled.
2. If private reporting is not available, open a public issue that asks the
   maintainers for a private contact channel. Do not include any
   vulnerability details in that issue.

Please include, where possible:

- a description of the issue and its potential impact,
- the affected document, design element or (in future) component,
- steps to reproduce or a minimal example, if applicable.

The maintainers will acknowledge reports on a best-effort basis. No response
time is guaranteed at this stage of the project.

## Design-level reports are welcome

Because the project is currently in the architecture phase, reports about
**design weaknesses** are in scope and valuable. Examples:

- a flow in which a model could reach verified completion without
  independent evidence,
- a path by which an effectful action could execute without a capability,
- a privilege expansion that does not require external authorization,
- an enforcement claim that extends beyond the boundary the runtime controls.

Public issues are acceptable for design discussion **only** when the
discussion does not disclose an exploitable vulnerability or sensitive
exploit details. Examples of acceptable public topics:

- an ambiguity in the documents,
- an inconsistency between documents,
- a missing open question.

If a report describes a way to actually bypass a control in any
implementation, deployment or integration, it must go through the private
process above. The same applies to steps, payloads or conditions that would
help someone exploit a weakness. If you are unsure, report privately.

## Handling of secrets in this repository

This repository must not contain credentials, keys, tokens, wallet material
or references to production infrastructure. If you find any, report it using
the process above.
