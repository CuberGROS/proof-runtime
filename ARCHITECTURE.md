# Architecture

> **Status: pre-alpha, Phase 0.** This document describes architectural
> boundaries only. Nothing described here is implemented yet.

This file is the **canonical entry point** to the Proof Runtime architecture.
It summarizes the proposed boundaries and identifies which document is
authoritative for each topic.

## Document roles

| Document                                                                     | Role                                                                                          |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `ARCHITECTURE.md` (this file)                                                | Canonical entry point and summary of the architecture boundaries                              |
| [docs/adr/0001-architecture-freeze.md](docs/adr/0001-architecture-freeze.md) | Decision record: what is proposed for freeze, terms under review, open questions              |
| [docs/invariants.md](docs/invariants.md)                                     | Detailed specification of the invariants: wording, meaning and enforcement scope             |
| [docs/architecture/overview.md](docs/architecture/overview.md)               | Explanatory narrative: plane responsibilities, action lifecycle, integration grades, diagrams |
| [spec/README.md](spec/README.md)                                             | Protocol requirements and the questions that block their specification                        |

Other documents summarize these sources and link to them rather than
restating them. If two documents disagree, treat it as a defect and raise it
for review. Until the defect is fixed, ADR 0001 governs what is decided, and
`docs/invariants.md` governs the wording of the invariants.

## Decision status

The boundaries below are **proposed for freeze** in
[ADR 0001](docs/adr/0001-architecture-freeze.md), whose status is
**Proposed**. They become frozen only when a maintainer explicitly accepts
the ADR.

- **While the ADR is Proposed:** changes go through review as amendments to
  ADR 0001.
- **After acceptance:** changes require a new ADR.

## Four planes

| Plane     | Components                                      |
| --------- | ----------------------------------------------- |
| CONTROL   | Identity, Policy, Capability, Risk, Approval    |
| EXECUTION | Action, Transaction, Sandbox, Secrets, Effects  |
| STATE     | Task Capsule, Checkpoint, Context, Memory       |
| TRUST     | Evidence, Claims, Verification, Receipts, Audit |

The responsibilities of each plane are described in
[overview.md § The four planes](docs/architecture/overview.md#the-four-planes).

## Four core protocols

- **Task Capsule** (STATE)
- **Action IR** (EXECUTION)
- **Capability Manifest** (CONTROL)
- **Evidence Receipt** (TRUST)

Each protocol must be **model-neutral**, **host-neutral**, **versioned** and
**extensible**. Field-level definitions are intentionally deferred. Intended
roles, requirements and blocking questions are in
[spec/README.md](spec/README.md).

## Invariants

The ten invariants are specified in [docs/invariants.md](docs/invariants.md).
They include the rule that a model must never directly transition an
executing task to verified completion.

## Integration grades

The runtime distinguishes **Observer**, **Integrated** and **Managed**
integration grades. Records must identify the integration grade and the
actual enforcement boundary under which each action ran. The runtime never
claims an enforcement guarantee outside the boundary it actually controls.
See [overview.md § Integration grades](docs/architecture/overview.md#integration-grades).

## Design principles

- **Neutrality.** The core protocols do not depend on a specific model
  provider, agent harness, programming language or industry.
- **Separation of proposal and authority.** Models propose; authority comes
  from the CONTROL plane and from external principals.
- **Separation of claim and fact.** Claims are inputs to verification, never
  outcomes of it.
- **Bounded guarantees.** Every guarantee is stated together with the
  boundary in which it holds.
- **Explicit uncertainty.** Unresolved decisions are recorded as open
  questions rather than filled in with assumptions.
