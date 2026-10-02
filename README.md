# Proof Runtime

Vendor-neutral execution and trust infrastructure for AI agents.

> **Status: pre-alpha.** This repository currently contains architecture
> documentation only. There is no runtime implementation, no published
> protocol schema, and no release. Nothing here should be relied on for
> production use, and no security, performance or correctness properties are
> claimed for any implementation.

## What Proof Runtime is for

Proof Runtime aims to give AI agents a common substrate for:

- **portable task state** that can move across models and hosts,
- **explicit capabilities** that bound what an agent is allowed to do,
- **governed execution** of effectful actions under policy and approval,
- **evidence-backed verification** of what actually happened, and
- **recovery** when execution fails or is interrupted.

The runtime is designed to be independent of any particular model provider,
agent harness, programming language or industry. Software engineering is the
first intended proving ground; the core protocols are meant to stay
industry-neutral.

## What "proof" means here

In this project, **proof** means **evidence-backed, machine-verifiable
execution records**: structured records that tie a task's claimed outcome to
the evidence collected while it executed, in a form that a machine can check.

It does **not** mean formal correctness proofs for arbitrary AI outputs. Proof
Runtime does not claim to establish that a model's output is correct, safe or
optimal. It aims to establish what was authorized, what was executed, what
evidence was collected, and which claims that evidence does or does not
support.

## Architecture at a glance

The architecture is organized into four planes and four core protocols. See
[ARCHITECTURE.md](ARCHITECTURE.md) and
[docs/architecture/overview.md](docs/architecture/overview.md) for details and
diagrams.

| Plane     | Responsibilities                                |
| --------- | ----------------------------------------------- |
| CONTROL   | Identity, Policy, Capability, Risk, Approval    |
| EXECUTION | Action, Transaction, Sandbox, Secrets, Effects  |
| STATE     | Task Capsule, Checkpoint, Context, Memory       |
| TRUST     | Evidence, Claims, Verification, Receipts, Audit |

| Protocol            | Purpose (summary)                                        |
| ------------------- | -------------------------------------------------------- |
| Task Capsule        | Portable representation of a task and its state          |
| Action IR           | Host-neutral representation of a proposed action         |
| Capability Manifest | Explicit statement of what an actor is permitted to do   |
| Evidence Receipt    | Verifiable record linking execution, evidence and claims |

All four protocols are required to be model-neutral, host-neutral, versioned
and extensible. Their fields are **not yet specified**; see
[spec/README.md](spec/README.md).

## Core invariants

The architecture is anchored by a set of non-negotiable invariants, including:

- `MODEL != AUTHORITY`
- `MODEL CLAIM != VERIFIED FACT`
- `NO EVIDENCE -> NO VERIFIED COMPLETION`
- `NO CAPABILITY -> NO EFFECTFUL ACTION`

The full list, with rationale and scope, is in
[docs/invariants.md](docs/invariants.md).

## Integration grades

Proof Runtime distinguishes three grades of integration with a host:
**Observer**, **Integrated** and **Managed**. Each grade has a different
enforcement boundary, and the runtime claims enforcement only inside the
boundary it actually controls. See
[docs/architecture/overview.md](docs/architecture/overview.md#integration-grades).

## Repository layout

```text
.
├── ARCHITECTURE.md      Entry point to the architecture
├── CONTRIBUTING.md      How to contribute during Phase 0
├── LICENSE              Apache License 2.0
├── NOTICE               Attribution notice
├── README.md            This file
├── SECURITY.md          Security reporting and scope
├── docs/
│   ├── adr/             Architecture Decision Records
│   ├── architecture/    Architecture narrative and diagrams
│   └── invariants.md    Non-negotiable invariants
└── spec/                Protocol specifications (not yet written)
```

## Roadmap status

- **Phase 0 (current):** repository foundation and architecture
  documentation. No runtime code.
- Later phases (protocol specification, reference implementation) begin only
  after Phase 0 is reviewed and explicitly authorized.

Open architectural questions are tracked in
[docs/adr/0001-architecture-freeze.md](docs/adr/0001-architecture-freeze.md#open-questions).

## Contributing and security

- Contribution guidelines: [CONTRIBUTING.md](CONTRIBUTING.md)
- Security reporting: [SECURITY.md](SECURITY.md)

## License

Licensed under the [Apache License, Version 2.0](LICENSE). See also
[NOTICE](NOTICE).
