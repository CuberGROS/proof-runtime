# ADR 0001: Architecture Freeze for v0.1 Foundation

- **Status:** Proposed (pending review)
- **Date:** 2026-10-02
- **Phase:** 0 — repository foundation and architecture documentation

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

The following are frozen for the v0.1 foundation. Any change requires a new
ADR that supersedes the relevant part of this one.

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

The ten invariants in [docs/invariants.md](../invariants.md) are
non-negotiable, including the rule that a model must never directly
transition an executing task to verified completion.

### 4. Integration grades

Three integration grades are recognized: **Observer**, **Integrated** and
**Managed**. They are defined in
[docs/architecture/overview.md](../architecture/overview.md#integration-grades).
The runtime claims enforcement only within the boundary it actually controls.

### 5. Meaning of "proof"

"Proof" means evidence-backed, machine-verifiable execution records. It does
not mean formal correctness proofs for arbitrary AI outputs.

## Consequences

- Later specification work must fit within the four planes and four
  protocols. If it cannot, a new ADR is required.
- Documentation and future code must state the enforcement boundary of any
  guarantee they describe.
- Several fundamental technical decisions remain open (below). They must be
  resolved, each through its own ADR, before the affected protocol is
  specified.
- No runtime implementation is authorized by this ADR.

## Terms identified for review

The documentation uses the following terms to describe the architecture.
They are **not** listed among the frozen plane components or protocols. They
are identified here so reviewers can decide whether any should become
explicit primitives, be renamed, or stay purely descriptive.

| Term                   | Where used                     | Current treatment                                                            |
| ---------------------- | ------------------------------ | ---------------------------------------------------------------------------- |
| Model / agent harness  | overview, invariants           | Descriptive: source of proposals and claims, outside the runtime's authority |
| Host                   | overview, invariants           | Descriptive: environment in which actions take effect                        |
| Principal              | overview                       | Descriptive: identity with authority; relates to CONTROL › Identity          |
| External approver      | overview, invariants           | Descriptive: principal giving authorization; relates to CONTROL › Approval   |
| Verifier               | overview                       | Descriptive: function within TRUST › Verification                            |
| Enforcement boundary   | overview, invariants, SECURITY | Descriptive: the region the runtime actually controls                        |
| Integration grade      | all architecture docs          | Specified by the Phase 0 brief; not a plane component                        |
| Effectful action       | invariants                     | Used by I7; the precise definition is an open question                       |
| Verified completion    | invariants, overview           | Task outcome; the full lifecycle state set is an open question               |
| Host-attested evidence | overview, invariants           | Descriptive: evidence whose provenance is the host, not the runtime          |

## Open questions

These questions are unresolved. Each should be settled by its own ADR or
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
- **OQ-13 Secrets handling.** How are secrets made available to authorized
  actions without exposing them to model-visible context, evidence or
  receipts?

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

- **OQ-17 Verification sufficiency.** What evidence is sufficient to verify a
  given kind of claim? How are partial verification and conflicting evidence
  represented?
- **OQ-18 Verifier trust.** How is a verifier's own trustworthiness
  established? Can a model act as a verifier, and under what constraints,
  given `MODEL CLAIM != VERIFIED FACT`?
- **OQ-19 Evidence storage, retention and privacy.** Where is evidence
  stored, for how long, and how is sensitive evidence redacted without
  breaking verifiability?
- **OQ-20 Audit integrity.** Should the audit log be tamper-evident (for
  example, append-only with hash chaining), and who can read it?
- **OQ-21 Recording integration grade.** How do Evidence Receipts express the
  integration grade and enforcement boundary under which evidence was
  collected?

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
