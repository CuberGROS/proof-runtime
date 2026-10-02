# ADR 0001: Architecture Freeze for v0.1 Foundation

- **Status:** Accepted
- **Date:** 2026-10-02
- **Accepted:** 2026-10-02, by the repository owner and maintainer
  (@CuberGROS)
- **Phase:** 0 — repository foundation and architecture documentation

This ADR was proposed and reviewed in pull request #1. The repository owner
and maintainer explicitly accepted it, and that acceptance is recorded in
this change.

## Context

Proof Runtime is intended to be vendor-neutral execution and trust
infrastructure for AI agents. Before any protocol specification or
implementation begins, the project needs stable top-level boundaries. Then
later work can be reviewed against them, and scope creep or premature
primitives can be identified.

The first proving ground is expected to be software engineering. The core
protocols must remain neutral with respect to model provider, agent harness,
host, programming language and industry.

## Decision

The following are frozen for the v0.1 foundation. Any change to them requires
a new ADR that supersedes the relevant part of this one.

### 1. Four planes

| Plane     | Components                                      |
| --------- | ----------------------------------------------- |
| CONTROL   | Identity, Policy, Capability, Risk, Approval    |
| EXECUTION | Action, Transaction, Sandbox, Secrets, Effects  |
| STATE     | Task Capsule, Checkpoint, Context, Memory       |
| TRUST     | Evidence, Claims, Verification, Receipts, Audit |

### 2. Four core protocols

- Task Capsule
- Action IR
- Capability Manifest
- Evidence Receipt

Each must be model-neutral, host-neutral, versioned and extensible. This ADR
deliberately does **not** define their fields, encodings or schemas.

### 3. Invariants

Ten invariants are adopted. [docs/invariants.md](../invariants.md) is their
detailed specification. They include the rule that a model must never
directly transition an executing task to verified completion.

### 4. Integration grades

Three integration grades are recognized: **Observer**, **Integrated** and
**Managed**. They are described in
[docs/architecture/overview.md](../architecture/overview.md#integration-grades).
Records must identify the integration grade and the actual enforcement
boundary under which each action ran. The runtime claims enforcement only
within the boundary it actually controls.

### 5. Meaning of "proof"

"Proof" means evidence-backed, machine-verifiable execution records. It does
not mean formal correctness proofs for arbitrary AI outputs.

## Consequences

- Later specification work must fit within the four planes and four
  protocols. If it cannot, it requires a new ADR.
- Documentation and future code must state the enforcement boundary of any
  guarantee they describe.
- Several fundamental technical decisions remain open (below). Each must be
  resolved through its own ADR or specification decision before the
  affected protocol is specified.
- No runtime implementation is authorized by this ADR.

## Terms identified for review

The documentation uses the following terms to describe the architecture.
They are **not** among the plane components, protocols or invariants frozen
above, and this ADR does not promote them to architectural primitives. They
are listed here with working definitions. Promoting any of them to an
explicit primitive, or renaming one, requires a new ADR.

| Term                   | Working definition                                                                                    | Where used                            | Related                    |
| ---------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------- | -------------------------- |
| Model / agent harness  | Model: an AI model producing proposals and claims. Agent harness: the software that drives it and turns its output into proposed actions | overview, invariants                  | I1, I2                     |
| Host                   | The environment in which actions take effect                                                          | overview, invariants                  | I5                         |
| Principal              | A human or system identity on whose behalf, or with whose authority, a task runs                     | overview, invariants                  | CONTROL › Identity, OQ-5   |
| External approver      | A principal outside the requesting actor who authorizes privilege expansion or approves high-risk actions | overview, invariants                  | CONTROL › Approval, I8, I9 |
| Verifier               | The function that evaluates claims against evidence                                                   | overview                              | TRUST › Verification, OQ-18 |
| Enforcement boundary   | The region within which the runtime can actually prevent or constrain behavior                        | overview, invariants, SECURITY        | I5, OQ-21                  |
| Control point          | A specific mechanism a host exposes through which the runtime's decision is enforced for the operations that pass through it (Integrated grade) | overview, invariants                  | I5, OQ-21                  |
| Integration grade      | Observer, Integrated or Managed (defined in §4 of this ADR); not a plane component                    | all architecture documents            | I5                         |
| Effectful action       | An action with effects beyond the runtime's internal bookkeeping                                      | invariants                            | I7, OQ-10                  |
| Default deny           | Restatement of I7: an effectful action without a covering capability is denied; not a separate rule  | invariants                            | I7                         |
| Provenance             | The recorded origin of a piece of data or evidence (runtime, host, tool, model)                      | invariants, overview                  | I4, OQ-17                  |
| Attestation (attested) | A statement by a party about something the runtime did not itself observe or enforce; e.g. host-attested evidence | invariants, overview                  | I4, I5                     |
| Decision record        | The recorded result of a CONTROL-plane decision (denial, approval, rejection, authorization)         | overview                              | CONTROL, TRUST › Audit, OQ-6 |
| Verification outcome   | The result of evaluating claims against evidence; the set of possible outcomes is **not defined** here | invariants, overview                  | I6, OQ-17                  |
| Verified completion    | The task outcome that only a TRUST-plane verification backed by evidence can establish; the full lifecycle state set is not defined | invariants, overview                  | I6, OQ-14                  |

## Open questions

These questions are unresolved. Each must be resolved through its own ADR or
specification decision. They are numbered for reference only, not in order
of priority.

### Protocol foundations

- **OQ-1 Encoding and schema language.** Which serialization format(s) and
  schema language(s) will the protocols use?
- **OQ-2 Versioning and compatibility.** What versioning scheme, and what
  forward and backward compatibility rules, apply to each protocol?
- **OQ-3 Extensibility model.** How are extensions namespaced? How does a
  consumer distinguish extensions it may ignore from extensions it must
  understand?
- **OQ-4 Canonicalization and integrity.** How are protocol documents
  canonicalized, hashed and signed? Which parties sign which documents, and
  how are their keys managed and rotated?

### CONTROL

- **OQ-5 Identity model.** Which kinds of principals exist (human, service,
  agent, runtime, host)? How are they identified? How does the runtime
  integrate with external identity providers?
- **OQ-6 Policy language.** What language or engine expresses policy? How is
  policy versioned, and how is it bound to a decision record?
- **OQ-7 Capability semantics.** How are capabilities scoped, delegated,
  attenuated, expired and revoked?
- **OQ-8 Risk model.** Who or what classifies risk? What is an
  industry-neutral risk taxonomy? Can risk be reassessed during execution?
- **OQ-9 Approval semantics.** How is an approval bound to the exact action
  it approves? Do approvals expire? Are quorum or multi-party approvals
  supported?
- **OQ-10 Definition of "effectful".** Which actions count as effectful for
  invariant I7? In particular, how are reads of sensitive data treated?

### EXECUTION

- **OQ-11 Transaction semantics.** Where do transaction boundaries sit? How
  are compensating actions defined? What happens to irreversible effects, and
  what happens when compensation itself fails?
- **OQ-12 Sandbox requirements.** What isolation properties must a sandbox
  provide to support the Managed grade?
- **OQ-13 Secrets handling.** Raw secret values must not appear in any
  protocol document, including Action IR. So how are secrets referenced by
  and supplied to authorized actions, and how are they kept out of
  model-visible context, evidence and receipts?

### STATE

- **OQ-14 Task lifecycle.** What are the states and allowed transitions of a
  task, and which actor may perform each transition? The only fixed
  constraint is that a model cannot directly transition an executing task to
  verified completion.
- **OQ-15 Context vs. Memory.** Where exactly is the boundary between Context
  and Memory? How is memory provenance tracked?
- **OQ-16 Portability and checkpoints.** What must a Task Capsule carry to
  resume on a different model or host? How is a change of integration grade
  represented?

### TRUST

- **OQ-17 Verification outcomes and sufficiency.** What set of verification
  outcomes can a receipt express? What evidence is sufficient to verify a
  given kind of claim? How are partial verification, conflicting evidence
  and evidence provenance represented?
- **OQ-18 Verifier trust.** How is a verifier's own trustworthiness
  established? Can a model act as a verifier, and under what constraints,
  given `MODEL CLAIM != VERIFIED FACT`?
- **OQ-19 Evidence storage, retention and privacy.** Where is evidence
  stored, for how long, and how is sensitive evidence redacted without
  breaking verifiability?
- **OQ-20 Audit integrity.** Should the audit log be tamper-evident (for
  example, append-only with hash chaining), and who can read it?
- **OQ-21 Recording integration grade.** How do records and Evidence Receipts
  identify the integration grade, the actual enforcement boundary and, for
  the Integrated grade, the control points in effect?

### Project and process

- **OQ-22 Reference implementation language.** Which language will the first
  reference implementation use? Rust has been mentioned as a candidate but is
  not decided by this ADR. The protocols must stay language-neutral either
  way.
- **OQ-23 Relationship to existing standards.** Should the protocols align
  with, reuse or interoperate with existing work on supply-chain
  attestations, verifiable credentials, observability and agent tool
  protocols? No compatibility is claimed today.
- **OQ-24 Copyright holder and NOTICE.** `NOTICE` currently attributes
  copyright to "The Proof Runtime Contributors". The maintainers should
  confirm the intended copyright line.
- **OQ-25 Security contact.** `SECURITY.md` relies on GitHub private
  vulnerability reporting, which must be enabled on the repository. Should
  there also be a dedicated contact address?
