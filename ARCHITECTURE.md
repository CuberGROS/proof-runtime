# Architecture

> **Status: pre-alpha, Phase 0.** This document describes architectural
> boundaries only. Nothing described here is implemented yet.

This file is the entry point to the Proof Runtime architecture. It states the
frozen boundaries and points to the documents that hold the detail. If this
file and a linked document disagree, treat it as a defect and raise it for
review.

| Document                                                                     | Contents                                               |
| ---------------------------------------------------------------------------- | ------------------------------------------------------ |
| [docs/architecture/overview.md](docs/architecture/overview.md)               | Planes, protocols, flows, integration grades, diagrams |
| [docs/invariants.md](docs/invariants.md)                                     | Non-negotiable invariants with rationale and scope     |
| [docs/adr/0001-architecture-freeze.md](docs/adr/0001-architecture-freeze.md) | The freeze decision, review items and open questions   |
| [spec/README.md](spec/README.md)                                             | Requirements and status of the four core protocols     |

## Frozen boundaries

The following boundaries are fixed by
[ADR 0001](docs/adr/0001-architecture-freeze.md). Changing them requires a new
ADR.

### Four planes

| Plane     | Components                                      |
| --------- | ----------------------------------------------- |
| CONTROL   | Identity, Policy, Capability, Risk, Approval    |
| EXECUTION | Action, Transaction, Sandbox, Secrets, Effects  |
| STATE     | Task Capsule, Checkpoint, Context, Memory       |
| TRUST     | Evidence, Claims, Verification, Receipts, Audit |

### Four core protocols

- **Task Capsule**
- **Action IR**
- **Capability Manifest**
- **Evidence Receipt**

Each protocol must be **model-neutral**, **host-neutral**, **versioned** and
**extensible**. Field-level definitions are intentionally deferred; see
[spec/README.md](spec/README.md).

### Invariants

The ten invariants in [docs/invariants.md](docs/invariants.md) are
non-negotiable. In particular, a model must never directly transition an
executing task to verified completion.

### Integration grades

The runtime distinguishes **Observer**, **Integrated** and **Managed**
integration grades. It never claims an enforcement guarantee outside the
boundary it actually controls. See
[overview.md § Integration grades](docs/architecture/overview.md#integration-grades).

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
