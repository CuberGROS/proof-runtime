# Shared Protocol Design Constraints

> **Status: Accepted**, as part of
> [ADR 0002](../adr/0002-shared-protocol-foundations.md) (accepted
> 2026-10-02). These constraints bind all four core protocols. They restate no
> field definitions; none exist yet.

Each constraint applies to Task Capsule, Action IR, Capability Manifest and
Evidence Receipt alike. Where a constraint follows from an
[invariant](../invariants.md) or a decision in ADR 0002, the source is
named. A protocol specification that cannot meet a constraint needs a new
ADR. It must not quietly deviate.

## Structure and processing

- **C1. Self-describing documents.** Every document identifies its type and
  protocol version (D2.2), and lists its critical extensions (D3.4). A
  consumer can decide whether it may process a document before it
  interprets any content.
- **C2. Fail closed.** A consumer rejects a document, and records why, when
  any of the following holds:
  - it is not valid under the JSON profile (D1.2). This includes any number
    token, anywhere in the document, that is not an integer spelling within
    ±(2^53 − 1), checked lexically before any lossy conversion. Fraction
    parts, exponents and `-0` are all rejected,
  - it fails validation against its protocol's normative JSON Schema
    (D1.3),
  - it exceeds the resource limits (D1.4),
  - its type identifier and declared version disagree on the major
    version (D2.2),
  - its major version is unknown (D2.3),
  - for `0.x` protocols, the consumer has not explicitly implemented its
    exact `MAJOR.MINOR`. No cross-minor compatibility exception exists
    before `1.0` (D2.6),
  - it lists a critical extension the consumer does not understand, or its
    critical list contains duplicate or non-ASCII identifiers (D3.4),
  - its critical list names an identifier that is not present in its
    extensions element (a dangling critical identifier; D3.4). This applies
    even when the consumer understands that identifier, and it is a
    cross-field check that ordinary schema validation may not catch,
  - it carries a known critical-class extension without listing it as
    critical (D3.6),
  - a digest reference it relies on lacks a `sha256` entry, or does not
    match when recomputed (D4.4, D4.5).

  A rejected document is never partially processed for a security-relevant
  decision.
- **C3. Decision neutrality of ignorable content.** Anything a consumer is
  permitted to ignore must be decision-neutral. This covers non-critical
  extensions (D3.3) and, from `1.0`, core members added in a newer minor
  version (D2.4, D2.5). Ignoring such content therefore never changes a
  security-relevant decision, in either direction. Content that affects a
  decision, whether it restricts or permits, is critical or requires a new
  major version (D2.5, D3.5).
- **C4. Immutability, not indefinite retention.** A document referenced by
  digest is never modified while it is retained.
  - A correction, migration or redaction produces a new document, which
    **MUST reference its predecessor by the predecessor's content digest**
    (D2.7, D4.5). That is how provenance is preserved.
  - It **MUST NOT** use the predecessor's digest as its own content digest,
    present itself under that digest, or reuse any endorsement made over
    the predecessor (D4.5). Its own content digest is computed from its
    own canonical form.
  - Whether a document is retained, deleted or tombstoned is decided by the
    retention and privacy policy (OQ-19). Deletion is never an in-place
    edit.
- **C5. Digest references.** Cross-document and artifact references carry
  content digests that always include `sha256` (D4.3, D4.4).
  - A digest is accepted for a document only after it has been recomputed
    from the document actually held (D4.5).
  - A reference that cannot be resolved, whose target has been deleted or
    tombstoned, or whose digest does not match, makes the referring claim
    unverifiable. It is not silently skipped, and it is never presented as
    verified (D2.7).

## Authority and trust

- **C6. Proposal is not authority** (I1). An Action IR document describes a
  proposed action. Nothing in it, including fields copied from a policy or
  an approval, authorizes execution. Authorization exists only as a
  CONTROL-plane decision. That decision is a separate record, and it refers
  to the proposal by digest.
- **C7. Declaration is not authorization.** A Capability Manifest states
  which capabilities an actor holds and their scope. It is an input to
  CONTROL-plane decisions, and four things must be kept apart:
  - **Declaration:** what the Capability Manifest says.
  - **Authorization:** a CONTROL decision that a specific action is
    permitted.
  - **Policy decision:** the policy evaluation that produced that
    authorization.
  - **Enforcement:** whether anything actually prevented an unauthorized
    action, which depends on the integration grade.

  A protocol document must never let one stand in for another.
- **C8. Declaration is not enforcement** (I5). A Capability Manifest, or
  any capability or support statement from a host, does not show that the
  host enforces it. Enforcement claims are recorded with the integration
  grade and the actual enforcement boundary (ADR 0001 §4).
- **C9. State is not authority** (I3). A Task Capsule's state, context and
  memory never grant authority, even when they reference capabilities or
  past approvals. A reference is a pointer that CONTROL re-evaluates; it is
  not a grant.
- **C10. Claims, observations and enforcement stay distinct** (I2, I4, I5).
  Evidence Receipts must distinguish at least three kinds of fact:
  - **Observed:** recorded by the runtime from what it could itself see.
  - **Host-attested:** reported by a host or tool and not independently
    observed.
  - **Runtime-enforced:** something the runtime itself controlled, inside
    the enforcement boundary of the recorded integration grade.

  Model claims are a separate category from all three. Recording, hashing
  or (in future) signing never moves a fact between these categories
  (D4.6). The exact vocabulary and its encoding are open (OQ-17, OQ-21).
- **C11. Integrity is not correctness** (I2, I6). A matching digest shows
  only that content is unchanged (D4.1):
  - the canonical JSON data model, for a protocol document,
  - the exact bytes, for an opaque artifact.

  A future signature (D4.7) would show only that a key holder endorsed
  that content. Neither shows that the content
  is true, that its author was authorized, or that an action executed as
  described. v0.1 defines no signatures, and no v0.1 document may claim to
  be signed.

## Data hygiene

- **C12. No raw secrets.** No protocol document contains a raw secret value
  in any core member or extension. This includes credentials, tokens,
  private keys and session cookies.
  - Secrets are referenced by opaque handles whose design is OQ-13.
  - Producers must redact secrets before emitting a document. Consumers
    must not log or forward a secret they encounter.
  - Automated secret detection is best-effort and cannot guarantee absence;
    the obligation sits with producers.
- **C13. Industry neutrality** (D3.8). Core members use domain-neutral
  vocabulary. Software-engineering terms such as repositories, commits and
  builds appear only in extensions, even though software engineering is the
  first proving ground.
- **C14. Model and host neutrality** (ADR 0001 §2). Core members do not
  assume a model provider, prompt format, agent harness, operating system or
  cloud provider. Such details appear only as extension data or opaque,
  attributed values.
- **C15. Time is a claim.** Timestamps record what the producer asserts
  (D1.2). No constraint or verification may depend on a timestamp being
  accurate unless a trusted time source is defined (OQ-28).

## Guarantees and boundaries

- **C16. Bounded guarantees.** A protocol specification must not describe a
  document as preventing, enforcing or guaranteeing anything beyond the
  integration grade and enforcement boundary under which it is produced or
  consumed. Observer-grade and non-mediated Integrated-grade records
  describe decisions and observations, not prevention (see
  [overview.md § Action lifecycle](../architecture/overview.md#action-lifecycle)).
- **C17. A record of a decision is not a credential** (I1, I5). An Evidence
  Receipt, an Audit record, or any other document that describes an
  authorization decision never authorizes anything by being presented.
  - Authorization is established only by the CONTROL plane, for the exact
    proposal concerned (by digest), at the enforcement point that
    evaluates it.
  - A consumer MUST NOT accept a receipt, or a copy of a decision record, as
    permission to execute.
  - This holds whatever design is chosen for decision records (OQ-29).
