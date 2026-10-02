# Protocol Specifications

> **Status: not yet specified.** This directory holds no protocol
> definitions yet. No fields, schemas or encodings are defined, and no
> placeholder schemas are provided on purpose. Specification work begins
> only after Phase 0 is reviewed and a specification phase is explicitly
> authorized.

Proof Runtime defines four core protocols. Their existence and purpose are
frozen by [ADR 0001](../docs/adr/0001-architecture-freeze.md). Their contents
are not.

## The four core protocols

| Protocol            | Primary plane | Intended role                                                                 |
| ------------------- | ------------- | ----------------------------------------------------------------------------- |
| Task Capsule        | STATE         | Portable task identity, goal and state, usable across models and hosts        |
| Action IR           | EXECUTION     | Host-neutral intermediate representation of a proposed action                 |
| Capability Manifest | CONTROL       | Explicit statement of the capabilities an actor holds and their scope         |
| Evidence Receipt    | TRUST         | Machine-verifiable record linking actions, evidence, claims and verification  |

## Requirements common to all protocols

Every protocol must be:

- **Model-neutral.** It contains no assumptions specific to a model provider,
  model family or prompting format.
- **Host-neutral.** It contains no assumptions specific to an agent harness,
  operating system, cloud provider or execution environment.
- **Versioned.** Every document identifies the protocol version it conforms
  to.
- **Extensible.** Domain-specific or host-specific information can be added
  without changing the core protocol.

In addition, every protocol must preserve the
[invariants](../docs/invariants.md). In particular:

- A Task Capsule must not let a model transition a task to verified
  completion.
- An Action IR document is a proposal. It does not carry authority.
- A Capability Manifest cannot be expanded by the actor it describes.
- An Evidence Receipt must keep claims distinct from verified facts, and must
  make the enforcement boundary under which evidence was collected visible.

## Open questions blocking specification

The questions below must be resolved before field-level specification
begins. Full descriptions are in
[ADR 0001 § Open questions](../docs/adr/0001-architecture-freeze.md#open-questions).

| Protocol            | Blocking questions                |
| ------------------- | --------------------------------- |
| All                 | OQ-1, OQ-2, OQ-3, OQ-4            |
| Task Capsule        | OQ-14, OQ-15, OQ-16               |
| Action IR           | OQ-10, OQ-11                      |
| Capability Manifest | OQ-5, OQ-6, OQ-7, OQ-8, OQ-9      |
| Evidence Receipt    | OQ-17, OQ-18, OQ-19, OQ-20, OQ-21 |

## Planned layout

When specification is authorized, each protocol is expected to get its own
subdirectory (for example, `spec/task-capsule/`). Each one would contain a
normative specification document and, once an encoding is chosen (OQ-1),
machine-readable schemas and conformance examples. This layout is a proposal
and can change.
