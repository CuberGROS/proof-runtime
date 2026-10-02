# Invariants

> **Status: frozen by [ADR 0001](adr/0001-architecture-freeze.md) (status:
> Accepted).** This document is the detailed specification of the
> invariants, and ADR 0001 records the decision to adopt them. Changing,
> removing or weakening an invariant requires a new ADR.

These invariants are non-negotiable design constraints. Every protocol,
component and integration must preserve them. Where an implementation cannot
enforce an invariant (for example, because execution happens outside the
runtime's control), it must not claim to, and its records must make that
limitation visible. See
[Integration grades](architecture/overview.md#integration-grades).

## Notation

- `A != B` means A must never be treated as, or silently promoted to, B.
- `A -> B` means whenever A holds, B is required.

## Summary

| #   | Invariant                                        |
| --- | ------------------------------------------------ |
| I1  | `MODEL != AUTHORITY`                             |
| I2  | `MODEL CLAIM != VERIFIED FACT`                   |
| I3  | `MEMORY != POLICY`                               |
| I4  | `TOOL OUTPUT != TRUSTED FACT`                    |
| I5  | `HOST SUPPORT != ENFORCEMENT`                    |
| I6  | `NO EVIDENCE -> NO VERIFIED COMPLETION`          |
| I7  | `NO CAPABILITY -> NO EFFECTFUL ACTION`           |
| I8  | `PRIVILEGE EXPANSION -> EXTERNAL AUTHORIZATION`  |
| I9  | `HIGH-RISK ACTION -> POLICY / APPROVAL`          |
| I10 | `FAILED TRANSACTION -> ROLLBACK OR COMPENSATION` |

Derived rule (follows from I1, I2 and I6):

> **A model must never directly transition an executing task to verified
> completion.** A model may claim completion. Only verification in the TRUST
> plane, backed by evidence, can establish verified completion.

## Details

### I1 — `MODEL != AUTHORITY`

**Meaning.** A model's output is a proposal. It never grants permission,
approves an action, or changes policy by itself.

**Rationale.** Model behavior is not fully predictable and can be steered by
its inputs. Authority has to originate from principals and policies that
the model cannot rewrite.

**Implication.** Every effectful action a model proposes is evaluated by the
CONTROL plane before execution.

### I2 — `MODEL CLAIM != VERIFIED FACT`

**Meaning.** Statements a model makes about the world or about its own work
("the tests pass", "the file was deleted") are recorded as **claims**. They
are inputs to verification, not results of it.

**Rationale.** Models can be wrong, stale or manipulated, and a claim
carries no evidence by itself.

**Implication.** The TRUST plane keeps claims and verified facts distinct.
A claim becomes a verified fact only through verification against evidence.

### I3 — `MEMORY != POLICY`

**Meaning.** Content in Context or Memory (STATE plane) never acts as policy,
capability or approval, even if it describes one.

**Rationale.** Memory can be written by models, by tools and by untrusted
inputs. If memory could act as policy, anything that writes to memory could
escalate privileges.

**Implication.** Policy, capability and approval decisions come only from the
CONTROL plane.

### I4 — `TOOL OUTPUT != TRUSTED FACT`

**Meaning.** Output returned by a tool, API or external system is data with a
recorded provenance. It is not trusted by default.

**Rationale.** Tools can fail, return partial results, or be compromised, and
their output can contain injected instructions.

**Implication.** Tool output may become evidence. How much weight it carries
in verification depends on its provenance, and it never acts as an
instruction to the runtime.

### I5 — `HOST SUPPORT != ENFORCEMENT`

**Meaning.** A host saying it supports a control (for example, "honors
capability denials") is not the same as the runtime enforcing that control.

**Rationale.** The runtime can only guarantee what happens inside a boundary
it controls. Behavior inside a host that the runtime does not control is, at
best, attested by that host.

**Implication.** Records must identify the integration grade and the actual
enforcement boundary under which each action ran. The runtime makes no
enforcement claim outside that boundary.

### I6 — `NO EVIDENCE -> NO VERIFIED COMPLETION`

**Meaning.** A task, or any claim about it, can be marked verified only if
evidence supports it.

**Rationale.** Without evidence, "verified" would mean "asserted".

**Implication.** When evidence is missing or does not support a claim, the
outcome must not be "verified". How other outcomes are represented is an
[open question](adr/0001-architecture-freeze.md#open-questions) (OQ-17).

### I7 — `NO CAPABILITY -> NO EFFECTFUL ACTION`

**Meaning.** An action with effects outside the runtime's internal
bookkeeping (writing files, calling external services, spending resources,
sending messages) needs an explicit capability that covers it.

**Rationale.** Capabilities make the scope of permitted effects explicit,
reviewable and auditable.

**Implication.** An effectful action without a covering capability is
denied. This "default deny" is a restatement of I7, not a separate rule.
Which actions count as effectful is an
[open question](adr/0001-architecture-freeze.md#open-questions) (OQ-10).

### I8 — `PRIVILEGE EXPANSION -> EXTERNAL AUTHORIZATION`

**Meaning.** Any expansion of what an actor may do (a new capability, a wider
scope, longer validity) must be authorized by a principal or policy outside
the actor requesting it.

**Rationale.** An actor that can expand its own privileges has no effective
bounds.

**Implication.** A model cannot grant itself capabilities. Requests for
expansion go to the CONTROL plane and through external authorization.

### I9 — `HIGH-RISK ACTION -> POLICY / APPROVAL`

**Meaning.** An action classified as high-risk runs only if policy explicitly
permits it, or an authorized approver approves it, as policy requires.

**Rationale.** Some actions do enough damage, or are hard enough to reverse,
that a capability alone is not sufficient.

**Implication.** The CONTROL plane assesses risk before execution. The risk
model is an [open question](adr/0001-architecture-freeze.md#open-questions)
(OQ-8).

### I10 — `FAILED TRANSACTION -> ROLLBACK OR COMPENSATION`

**Meaning.** When a transaction fails, its effects are either rolled back or
addressed by a defined compensating action.

**Rationale.** Partial, unrecorded effects make recovery and verification
unreliable.

**Implication.**

- I10 is satisfied only when the failed transaction's effects are actually
  rolled back, or addressed by a compensating action that succeeds.
  Attempting rollback or compensation is not the same as recovering.
- Every rollback or compensation attempt, and its outcome, is recorded as
  evidence.
- If both rollback and compensation fail, the records must identify the
  unrecovered effects. They must explicitly classify the situation as an
  **unresolved violation of I10**, and it is never hidden. While the
  violation is unresolved, neither successful recovery nor verified
  completion may be claimed for the affected task.
- Detailed transaction semantics, including irreversible effects and
  failed compensation, are an
  [open question](adr/0001-architecture-freeze.md#open-questions) (OQ-11).

## Enforcement scope

How strongly an invariant about execution can be upheld depends on the
integration grade (see
[overview.md § Integration grades](architecture/overview.md#integration-grades)):

| Grade      | What the runtime can do for an invariant                                                                                                                                                                                                                  |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Observer   | Detect and record apparent violations it has evidence of. It cannot prevent them.                                                                                                                                                                         |
| Integrated | Enforce its decisions through the specific control points the host exposes, for the operations that actually pass through them. For anything else it can only issue and record decisions; whether the host honors them is host-attested, not enforced. |
| Managed    | Enforce, within the execution boundary the runtime itself controls.                                                                                                                                                                                      |

Some invariants govern what the runtime records, claims and concludes: I2,
I3, I4, I5, I6 and the derived rule. These apply at **every** grade, because
the runtime's own records and verification outcomes are always within its
control.
