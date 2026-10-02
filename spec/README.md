# Protocol Specifications

> **Status: not yet specified.** This directory holds no protocol
> definitions yet. No fields, schemas or encodings are defined, and no
> placeholder schemas are provided on purpose. Specification work begins
> only after Phase 0 is reviewed and a specification phase is explicitly
> authorized.

Proof Runtime defines four core protocols. Their existence and purpose are
proposed for freeze in [ADR 0001](../docs/adr/0001-architecture-freeze.md),
whose status is **Proposed**. Their contents are not yet defined.

## The four core protocols

| Protocol            | Primary plane | Intended role                                                                |
| ------------------- | ------------- | ---------------------------------------------------------------------------- |
| Task Capsule        | STATE         | Portable task identity, goal and state, usable across models and hosts       |
| Action IR           | EXECUTION     | Host-neutral intermediate representation of a proposed action                |
| Capability Manifest | CONTROL       | Explicit statement of the capabilities an actor holds and their scope        |
| Evidence Receipt    | TRUST         | Machine-verifiable record linking actions, evidence, claims and verification |

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
- An Evidence Receipt must keep claims distinct from verified facts. It must
  identify the integration grade and the actual enforcement boundary under
  which its evidence was collected.
- No protocol document, including Action IR, may contain raw secret values.
  How authorized actions obtain secrets is an open question (OQ-13).

## Open questions blocking specification

The questions below must be resolved before field-level specification of
the listed protocol begins. Full descriptions are in
[ADR 0001 § Open questions](../docs/adr/0001-architecture-freeze.md#open-questions).

| Protocol            | Blocking questions                             |
| ------------------- | ---------------------------------------------- |
| All                 | OQ-1, OQ-2, OQ-3, OQ-4, OQ-23                  |
| Task Capsule        | OQ-5, OQ-13, OQ-14, OQ-15, OQ-16               |
| Action IR           | OQ-5, OQ-8, OQ-9, OQ-10, OQ-11, OQ-13          |
| Capability Manifest | OQ-5, OQ-6, OQ-7, OQ-8, OQ-9, OQ-10            |
| Evidence Receipt    | OQ-5, OQ-13, OQ-17, OQ-18, OQ-19, OQ-20, OQ-21 |

Why each dependency exists:

- **OQ-23 (existing standards)** blocks every protocol. Deciding whether to
  reuse or interoperate with existing standards after fields are designed
  could force incompatible changes. Listing it here classifies it as a
  dependency only; it does not resolve the question.
- **OQ-5 (identity)** blocks every protocol that names an actor: the
  principal of a task, the proposer of an action, the holder of a
  capability, the signer of a receipt.
- **OQ-8 (risk)** and **OQ-10 (effectful)** block Action IR and Capability
  Manifest. Capabilities and risk assessment both apply to proposed
  actions, so these definitions must agree.
- **OQ-9 (approval)** blocks Action IR because an approval must be bound to
  the exact action it approves.
- **OQ-13 (secrets)** blocks:
  - Action IR, which must not carry raw secret values,
  - Task Capsule, whose state must be portable without leaking secrets,
  - Evidence Receipt, whose evidence must be verifiable without exposing
    secrets.

## Planned layout

When specification is authorized, each protocol is expected to get its own
subdirectory (for example, `spec/task-capsule/`). Each one would contain a
normative specification document and, once an encoding is chosen (OQ-1),
machine-readable schemas and conformance examples. This layout is a proposal
and can change.
