# ADR 0003: Shared Protocol Blocker Decisions

- **Status:** Proposed
- **Date:** 2026-10-02
- **Phase:** 1B — shared blocker decisions
- **Proposes resolutions for:** OQ-26 and OQ-29 from
  [ADR 0002](0002-shared-protocol-foundations.md#new-open-questions), and
  OQ-5 and OQ-27 (OQ-5 from
  [ADR 0001](0001-architecture-freeze.md#open-questions), OQ-27 from ADR
  0002)
- **Does not change:** ADR 0001, ADR 0002, the four planes, the four core
  protocols, the ten invariants or the three integration grades

This ADR is **Proposed**. It resolves nothing until it is accepted.

- **Four independent decisions.** D6 (OQ-26), D7 (OQ-27), D8 (OQ-5) and
  D9 (OQ-29) each carry their own status, rationale, alternatives, security
  implications, dependencies and acceptance criteria. The repository owner
  may accept, amend or reject each one separately.
- **No question is resolved until accepted.** An open question is resolved
  only when the repository owner explicitly accepts the decision that
  addresses it, and that acceptance is recorded in this ADR. Until then,
  every question listed above remains open, and the
  [dependency map](../protocols/dependency-map.md) continues to show it as
  blocking.
- **Owner decisions.** Where a choice belongs to the owner, the decision
  says so under **Owner decision required**. In particular, D6 does not
  select a domain, register a namespace or fix the type-URI authority.
- **Numbering.** Decisions are numbered D6 to D9, continuing ADR 0002's D1
  to D5, so that a decision number such as "D7.2" is unambiguous across the
  two ADRs.

Supporting analysis:
[shared-blockers-analysis.md](../protocols/shared-blockers-analysis.md).
It gives the requirements, full alternatives, evidence, proposed
conformance cases and source verification for each decision.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in the
Decision section are to be interpreted as described in BCP 14 (RFC 2119 and
RFC 8174) when, and only when, they appear in all capitals. They take
effect only for a decision that has been accepted.

## Context

ADR 0002 settled the shared encoding, versioning, extension and integrity
conventions. It left four questions that block field-level specification:

- **OQ-26** and **OQ-27** block all four protocols. Every document needs a
  type identifier (D2.2) and stated resource limits (D1.4).
- **OQ-5** blocks every protocol that names an actor.
- **OQ-29** blocks the Evidence Receipt, and blocks Action IR
  conditionally.

ADR 0002 also recorded a constraint: OQ-29 must not be resolved by adding a
fifth core protocol without a new ADR amending ADR 0001.

## Boundaries preserved by every decision

The following hold for every decision in this ADR and are not changed by
it:

- Action IR is a proposal, not authority (I1, C6).
- An Evidence Receipt is not an authorization credential (C17).
- A decision record does not grant permission merely by being presented
  (C17). Neither does an identity recorded in it (D8.11).
- Authorization is evaluated by CONTROL for the exact proposed action, and
  enforced only at an actual controlled execution boundary (C7, C16,
  ADR 0001 §4).
- Identifying a model or an agent never grants authority (I1, I8).
- No new core protocol, plane component or invariant is introduced.
- No protocol fields, JSON Schemas, runtime code, SDKs or adapters are
  defined.

## Decision

### D6. Identifier namespace (OQ-26)

**Status: Proposed.** Not accepted. OQ-26 remains open.

1. **Authority.** The core namespace root (`{root}`) MUST be under an
   authority that the project itself controls, as OQ-26 requires
   (ADR 0002). Two forms meet that requirement:
   - **A (proposed):** an `https` URI under a DNS domain that the project
     controls. `{root}` is `https://{domain}/{base}`.
   - **C (alternative):** a `tag:` URI (RFC 4151) whose tagging entity is a
     DNS domain the project controls, or a role email address at such a
     domain, on the minting date. `{root}` is
     `tag:{domain},{YYYY-MM-DD}:{base}`. It meets the control requirement
     only under the conditions in D6.8.

   **Owner decision required:** this ADR does not choose or register a
   domain, does not choose a minting date and does not fix `{root}`. The
   owner chooses A or C, and records `{domain}`, `{base}` and, under C, the
   date.

   **Options that cannot resolve OQ-26 under this ADR:**
   - A w3id.org identifier (analysis option B) places the URI authority
     under a third party's governance, so it does not meet the control
     requirement. Adopting it would need an explicit governance decision by
     the owner and a new ADR that changes that requirement.
   - A GitHub-hosted URL, a `urn:uuid:` URI and a `tag:` URI under a
     personal email address are excluded for the same reason.
   - A formal URN namespace is not recommended now.

   Until the owner records the choice and `{root}` in this ADR, no core
   identifier is final.
2. **Identifiers are names, not locators.** No conformant processing step
   dereferences an identifier: consumers and validators MUST NOT
   dereference a type identifier, an extension identifier or a schema
   `$id` while processing a document. Dereferencing for human
   documentation is outside processing. This extends the offline rule of
   D1.3 to every identifier.
3. **Canonical spelling of core identifiers.** Every identifier minted in
   the core namespace MUST:
   - be ASCII and at most 2,048 bytes (D7.3),
   - under option A, use the scheme `https` in lowercase, with a
     lowercase DNS host, and no user information, port, query, fragment or
     percent-encoding,
   - under option C, use the scheme `tag` in lowercase, a lowercase DNS
     name or role email address, the full `YYYY-MM-DD` date form (the
     `YYYY` and `YYYY-MM` forms are not used), and no query, fragment or
     percent-encoding,
   - have no empty path segments, no `.` or `..` segments and no trailing
     `/`. Under option C, path segments are the `/`-separated parts of the
     tag's specific part,
   - use only the characters `a` to `z`, `0` to `9`, `-` and `.` in path
     segments.

   Extension identifiers outside the core namespace SHOULD follow the same
   rules. Comparison remains exact (D1.2). Consumers MUST NOT normalize an
   identifier before comparing it.
4. **Template.** Under whichever authority is chosen:
   - a core type identifier MUST have the form `{root}/{protocol}/v{MAJOR}`,
   - a core extension identifier MUST have the form
     `{root}/ext/{name}/v{MAJOR}`,
   - a schema `$id` SHOULD have the form
     `{root}/{protocol}/v{MAJOR}/schema/{MAJOR}.{MINOR}`.

   `{protocol}` is one of `task-capsule`, `action-ir`,
   `capability-manifest` and `evidence-receipt`. `{MAJOR}` and `{MINOR}`
   use the D2.2 component syntax. A form segment within a protocol, if the
   Evidence Receipt specification needs one under D9.2, has the form
   `{root}/evidence-receipt/{form}/v{MAJOR}`.
5. **Minting and recognition.**
   - Only an accepted ADR or an accepted protocol specification may mint an
     identifier in the core namespace.
   - Consumers MUST recognize core identifiers only by exact match against
     the identifiers bundled with them. They MUST NOT treat an identifier as
     core, known or trusted because it starts with `{root}`.
   - An identifier's authority confers no trust in, and no information
     about, the producer of a document that carries it. Producer identity
     is D8; authenticity is the OQ-4 residual.
6. **Durability safeguards** (operational, for option A, and for the
   domain behind option C until its minting date is fixed):
   - the domain SHOULD be registered to an organization or role account,
     not to an individual,
   - it SHOULD be renewed for several years at a time, with automatic
     renewal and a registrar transfer lock,
   - it SHOULD be used for nothing that could be hijacked independently of
     the namespace,
   - the registration and renewal arrangements SHOULD be recorded in the
     repository.
7. **Placeholder.** Until `{root}` is recorded, drafts MUST use
   `https://example.invalid/proof-runtime` as `{root}`. A published
   specification version MUST NOT contain the placeholder, and an
   implementation release MUST NOT bundle it as a recognized identifier.
8. **Conditions for option C.** A `tag:` root meets the control requirement
   only if all of the following hold:
   - the tagging entity is a DNS domain the project controls, or a role
     email address at such a domain. A personal email address is not
     eligible, because the individual, not the project, controls it,
   - the project controlled that domain at 00:00 UTC on the date in
     `{root}` (RFC 4151), and evidence of that control, such as the
     registration record, is kept in the repository,
   - exactly one tagging entity and one date are used for the core
     namespace, so `{root}` has one spelling (D6.3),
   - minting stays restricted as D6.5 requires.

   Under these conditions the namespace is fixed to the project at minting
   time. A later registrant of a lapsed domain is not entitled by RFC 4151
   to mint identifiers with that date. A lapse then affects only
   documentation discoverability, never ownership of existing identifiers
   or any consumer decision (D6.2).

**Rationale.** Under D6.2, losing control of a domain cannot change any
conformant consumer's decision. What remains is the social-engineering
risk that D6.6 reduces. Option A then fully meets the requirement that the
namespace be under the project's control, and matches established
practice for `https` identifiers. Option C also meets it under D6.8. It is
stronger against domain lapse but gives up discoverability. Option B does
not meet the control requirement, because a third party governs the
authority.

**Alternatives considered.** Options A to F. See
[analysis § OQ-26](../protocols/shared-blockers-analysis.md#oq-26-identifier-namespace).

**Security implications.**
- Fetching identifiers would hand control of consumers to whoever controls
  a host. D6.2 forbids it.
- Prefix-based trust would let anyone claim core status by writing a
  string. D6.5 forbids it.
- Spelling divergence fails closed for critical extensions. D6.3 removes it
  for core identifiers.
- Namespace ownership is not producer identity (D6.5).
- An authority the project does not control would let a third party
  redirect, deny or reassign the namespace. D6.1 excludes it.

**Unresolved dependencies.** The owner decision in D6.1. The form segment
in D9.2.

**Acceptance criteria.** D6 can be accepted, and OQ-26 marked resolved,
only when the owner has:
- chosen option A or C. Option B cannot resolve OQ-26 under this ADR; it
  would need a new ADR that changes the control requirement,
- recorded `{root}` in this ADR, and confirmed that the domain is
  registered to the chosen registrant. Under option C, the owner has also
  recorded the evidence of control on the minting date (D6.8),
- confirmed or amended D6.2 to D6.8.

The owner may accept D6.2 to D6.8 before choosing an authority. In that
case OQ-26 remains open until D6.1 is completed.

### D7. Protocol resource limits (OQ-27)

**Status: Proposed.** Not accepted. OQ-27 remains open.

1. **Two tiers.** There is one shared **ceiling**, independent of protocol
   (D7.3). Each protocol specification MUST state its own limit for every
   measure in D7.2, at or below the ceiling. Where a specification states
   no tighter value, the ceiling is that protocol's limit. This satisfies
   D1.4.
2. **Measures.** Each measure is defined exactly so that every language
   gives the same answer:
   - **Document size:** the number of bytes in the JSON text as received.
   - **Nesting depth:** the top-level value has depth 1. Each object or
     array nested inside another adds 1. Scalars add nothing. Extension
     data counts.
   - **String length:** the number of bytes in the UTF-8 encoding of the
     decoded string value, after escape processing. It applies to string
     values and to member names. Lengths MUST NOT be measured in UTF-16
     code units, in code points or in source bytes.
   - **Array length:** the number of elements.
   - **Object size:** the number of members.
   - **Total values:** the number of JSON values in the document,
     including the top-level value and every nested object, array, string,
     number, boolean and `null`. Member names are not values.
   - **Extensions:** the number of members of the extensions element (D3.1)
     and the number of entries in the critical list (D3.4).
   - **Digest-set entries:** the number of algorithm entries in a digest
     set (D4.4).

   A document is within a limit when its measure is less than or equal to
   it.
3. **Ceiling values.**

   | Measure | Ceiling |
   |---|---|
   | Document size | 1,048,576 bytes |
   | Nesting depth | 32 |
   | String length (string values) | 65,536 bytes |
   | Member-name length | 2,048 bytes |
   | Identifier length (type, extension and schema identifiers; each identity-reference component, D8.2) | 2,048 bytes |
   | Array length | 4,096 elements |
   | Object size | 1,024 members |
   | Total values | 100,000 |
   | Extensions per document | 64 |
   | Critical-list entries | 64 |
   | Digest-set entries | 8 |

   The D1.2 integer profile already limits a number token to 17 bytes.
   Consumers SHOULD reject a longer token lexically, before any conversion.
4. **Order of checks.** Consumers MUST:
   - read no more than the size ceiling plus one byte before rejecting an
     oversized document, and MUST NOT parse a document that exceeds it,
   - check every other ceiling, the D1.2 profile and duplicate member
     names, before schema validation, canonicalization, digest
     computation, reference resolution or any security-relevant use,
   - check the declared protocol's own limits once its type and version
     are known, and before schema validation.

   Consumers SHOULD check the ceiling in a single streaming pass. A
   non-streaming implementation conforms only if it enforces the size
   ceiling before parsing, and its parser cannot exhaust the stack on any
   document within the ceiling.
5. **Limits and versions.** A protocol's limits are fixed for each major
   version. Changing them requires a new major version (D2.3); D2.4 permits
   no such change in a minor version. Raising the ceiling requires a new
   ADR.
6. **Local stricter refusal.** A consumer MAY refuse a document that is
   within all limits for local resource reasons. It MUST then:
   - record the refusal as a local resource refusal, distinct from
     invalidity,
   - not process the document partially,
   - not report the document as invalid to other parties.
7. **Producers.** Producers MUST NOT emit documents that exceed the limits
   of the declared protocol version. A party that re-serializes a document
   MUST NOT emit a serialization that exceeds the size limit. It forwards
   the original bytes instead (D4.5).
8. **Reporting.** A rejection for exceeding a limit is recorded with the
   measure that was exceeded. Rejection logs SHOULD NOT copy more than a
   bounded excerpt of the rejected content.

**Rationale.** The ceiling must be independent of protocol, because a
consumer cannot know a document's type before tokenizing it. The values
are derived from observed parser defaults, from the stack depth needed by
recursive validators and canonicalizers, from algorithmic complexity bounds
and from the metadata-only nature of documents (D1.2). See the
justification for each row in
[analysis § OQ-27](../protocols/shared-blockers-analysis.md#oq-27-protocol-resource-limits).

**Alternatives considered.**
- L1, per-protocol limits only: unbounded first pass.
- L2, one ceiling only: does not meet D1.4.
- L4, producer-declared limits: circular.
- L5, consumer-configured limits only: inconsistent across consumers.

**Security implications.**
- Bounds memory, stack and CPU (quadratic checks, sorting, hashing) before
  any trust is established.
- Removes parser differentials by fixing byte-based measures.
- Bounds log amplification.
- Limits only ever cause rejection, so they cannot weaken an authorization
  decision.

**Unresolved dependencies.**
- Per-protocol values, which are set in each protocol specification.
- Artifact size and reference-resolution budgets, which belong to the
  Evidence Receipt specification and OQ-19. A reference left unresolved
  because a budget ran out is unverifiable, never verified (C5).

**Acceptance criteria.** D7 can be accepted, and OQ-27 marked resolved,
when the owner:
- confirms or amends each ceiling value in D7.3,
- confirms the measures in D7.2,
- confirms that limits change only with a new major version (D7.5),
- confirms the local-refusal rule (D7.6).

### D8. Identity model (OQ-5)

**Status: Proposed.** Not accepted. OQ-5 remains open.

1. **Distinct concepts.** The following are distinct, and a protocol
   document MUST NOT let one stand in for another:
   - entity,
   - identity reference,
   - credential,
   - authenticated principal,
   - principal,
   - actor,
   - agent,
   - model,
   - delegate,
   - capability holder,
   - approver,
   - producer,
   - signer,
   - verifier,
   - authorization.

   One entity may hold several roles. Holding one role never implies
   another. The meanings are those in
   [analysis § OQ-5, Distinctions](../protocols/shared-blockers-analysis.md#distinctions).
2. **Identity references.** An entity is identified by an identity
   reference: a pair `(issuer, subject)`.
   - The issuer is an absolute URI naming an identity system.
   - The subject is an opaque string assigned by that issuer.
   - Both are ASCII and at most 2,048 bytes each (D7.3).
   - Two references are equal only if both components are equal under
     exact comparison (D1.2). No case folding or other normalization is
     applied.
3. **Issuers.**
   - An issuer relied on for principals MUST NOT reassign a subject to a
     different entity.
   - Which issuers are trusted, and for which entities, is CONTROL
     configuration. It MUST NOT be established by the content of any
     protocol document.
4. **Entity kinds.**
   - Every entity has exactly one kind: `human`, `service` or `agent`.
   - Kind is established by CONTROL's registration or issuer
     configuration. It MUST NOT be taken from content produced by the
     entity itself.
   - A model is not an entity kind. Organizations are not entity kinds.
   - Runtime, host, verifier, approver and producer are roles, not kinds.
   - Adding a kind requires a new ADR.
5. **Identity provenance.** Every identity that CONTROL uses or records has
   exactly one provenance, fixed when CONTROL records it:
   - **authenticated:** CONTROL itself validated a credential for this
     interaction, by a stated method, at a stated boundary,
   - **trusted-attested:** asserted by an attester (a host, adapter or
     identity provider) that CONTROL's configuration explicitly names as
     trusted for that issuer, and received directly from that attester over
     a channel that CONTROL authenticates. An attestation relayed through
     the actor, the model, the proposal or any other document is not
     trusted-attested,
   - **self-asserted:** any other identity claim. This includes proposal
     content, model output, memory, document content, and attestations from
     an attester that CONTROL is not configured to trust. Records SHOULD
     note the source of the claim.

   Rules:
   - Only authenticated and trusted-attested identities MAY support a
     permit. Whether a trusted-attested identity can satisfy a
     capability-holder or approver check is governed by D8.9.
   - A self-asserted identity MUST NOT support a permit. It MAY be
     recorded, and it MAY lead only to denial or further restriction.
   - Provenance MUST NOT be upgraded. A record keeps the provenance the
     identity had when the decision was made. A later authentication of the
     same entity leads to a new decision and a new record. It never rewrites
     an earlier record.
   - Provenance governs what CONTROL may rely on when deciding. It creates
     no enforcement (D8.14).
6. **Identification never grants authority.** No entity is authorized by
   being identified, authenticated or described. Authorization is a CONTROL
   decision over the exact proposal (C6, C7), requiring:
   - a covering capability (I7),
   - policy or approval, where I8 or I9 requires it.

   Identity determines which capabilities and policies apply. It is not
   itself a grant.
7. **Agents and models are never authority.**
   - An entity of kind `agent` MUST NOT act as an approver for I8 or I9,
     grant or expand capabilities, or author policy.
   - Model descriptors (provider, name, version) and agent
     self-descriptions, such as an A2A Agent Card, are at most
     trusted-attested. They MUST NOT be used to permit or to expand any
     action.
   - A policy MAY use them only to further restrict. A restriction based on
     an unauthenticated descriptor MUST be recorded as giving no
     enforcement assurance.
   - Whether an agent may act as a verifier is left to OQ-18.
8. **Actor and principal.**
   - **Two attributions, always.** Every proposal carries two separate
     attributions, and both are always recorded:
     - its **actor**, the entity that submitted it,
     - its **principal**, the entity on whose behalf it acts.

     Separate attribution does not require different identity references.
   - **The actor comes from CONTROL.** The actor is the identity CONTROL
     established for the submitter under D8.5. It is never taken from the
     proposal's content. If the proposal claims a different actor, that
     claim is recorded as self-asserted.
   - **Direct action.** A human or service entity acting on its own behalf
     is both actor and principal. Both attributions then carry the same
     identity reference with the same provenance. No delegation is
     involved.
   - **Delegated action.** When the principal differs from the actor,
     CONTROL MUST NOT evaluate the proposal on the principal's behalf unless
     it has validated a delegation from that principal to that actor
     (OQ-7). The decision then records:
     - the actor's own identity reference,
     - the principal's identity reference,
     - a reference to the validated delegation.

     Without a validated delegation, the principal attribution is
     self-asserted, and CONTROL denies the proposal.
   - **No impersonation.** An actor MUST NOT be recorded or accepted under
     another entity's identity reference. "Actor equals principal" is valid
     only when the actor established under D8.5 is that principal.
   - **No expansion.**
     - In a delegated action, the available authority is bounded both by
       what the principal holds and by what the delegation grants (I8).
     - Capabilities the actor holds that do not derive from this delegation
       MUST NOT be combined with the principal's authority to authorize the
       proposal.
     - In a delegation chain, a hop contributes to authorization only if
       CONTROL validated that hop. Otherwise it is recorded as
       informational.
   - **Agents are not principals** (proposed; owner decision required). An
     entity of kind `agent` is never a principal. Every agent proposal is
     therefore a delegated action from a human or service principal. The
     alternative is set out in
     [analysis § PC-2](../protocols/shared-blockers-analysis.md#pc-2-may-an-agent-be-a-principal).
   - Delegation scope, attenuation, expiry and revocation are OQ-7.
9. **Capability holders and approvers.**
   - A Capability Manifest names its holder by identity reference.
     Possessing or presenting a Capability Manifest confers nothing (C7).
   - CONTROL considers a capability for a proposal only if:
     - in a direct action, the holder is the actor established under D8.5,
     - in a delegated action, the capability is the principal's and is
       available through a delegation validated under D8.8.
   - **Provenance for holder checks** (proposed; owner decision required;
     alternatives in
     [analysis § PC-1](../protocols/shared-blockers-analysis.md#pc-1-provenance-required-for-capability-holder-checks)).
     - By default, the identity matched against the holder MUST be
       authenticated.
     - A trusted-attested identity satisfies a holder check only where
       CONTROL's configuration explicitly allows that attester to do so for
       that issuer.
     - Without that explicit allowance, a trusted-attested identity cannot
       satisfy the check, and the proposal is denied for lack of a covering
       capability (I7).
     - The same rule applies when, in a delegated action, the actor is
       matched against the delegate named in the validated delegation.
     - The decision record states the provenance that was used.
   - **Approver identity.** An approver's identity (I8, I9) MUST be
     authenticated. A trusted-attested identity never satisfies an approval
     requirement. Other approval semantics remain OQ-9.
   - A self-asserted identity never satisfies a holder check or an approval
     requirement (D8.5).
10. **Signers (reserved).** When a later ADR defines signing (the OQ-4
    residual):
    - keys bind to identity references,
    - an identity is never defined by a key, and rotating a key does not
      change an identity,
    - a signature shows endorsement only, not authorization or correctness
      (D4.1).
11. **Recorded identity is not authentication.** An identity reference in a
    document is a statement by the document's producer. A consumer MUST NOT
    treat it as authenticating any entity.
12. **Privacy.**
    - Subjects SHOULD be opaque and pseudonymous.
    - Core identity references MUST NOT carry names, email addresses or
      other personal attributes beyond the subject itself.
    - Display data, if any, belongs in non-critical extensions, subject to
      OQ-19.
13. **External identity systems.** External identity systems connect
    through adapters (D5.3):
    - **OIDC:** `iss` maps to issuer and `sub` maps to subject.
    - **SPIFFE:** issuer is `spiffe://{trust-domain}`, and subject is the
      full SPIFFE ID.
    - Other systems need a specified mapping.

    An adapter's mapping counts as authenticated only if CONTROL validated
    the underlying credential itself. Otherwise it is trusted-attested if
    the adapter is configured as a trusted attester under D8.5, and
    self-asserted if it is not (D5.5).
14. **Recording is not enforcement.** The rules above govern what CONTROL
    may rely on when it decides, and what it records. They create no
    enforcement.
    - At the Observer grade, and for Integrated operations outside an
      exposed control point, every decision, permits included, is a record
      only (C16). It is recorded with the provenance actually available,
      which may be only trusted-attested or self-asserted. D8.5 and D8.9
      still apply: a permit, even one that is only recorded, never rests on
      a self-asserted identity.
    - A permit takes effect only at a controlled execution boundary, where
      the enforcement point receives it directly from CONTROL (D9.4, D9.5).
    - No provenance, authenticated included, turns a record into
      enforcement, or into authorization for any later execution.

**Rationale.**
- Issuer-scoped references match how OIDC and SPIFFE already scope
  identity, and make explicit which issuer is relied on.
- Three kinds are enough to state the one structural rule that matters most
  (agents never approve), without confusing kinds with roles.
- Provenance categories mirror C10, so identity cannot be laundered from
  self-asserted to authenticated.

**Alternatives considered.**
- Identifier form: a single global URI, runtime-local identifiers,
  key-as-identity, DIDs or VCs.
- Kinds: fine-grained kinds, or no kinds.

See
[analysis § OQ-5](../protocols/shared-blockers-analysis.md#oq-5-identity-model).

**Security implications.** This decision addresses:
- self-asserted identity, relayed attestations, provenance upgrades and
  kind spoofing (D8.4, D8.5),
- impersonation, and the confused deputy created by combining an
  actor's own capabilities with a principal's (D8.8),
- unverified delegation chains (D8.8), avoiding the gap left by RFC 8693's
  informational-only nested actors,
- cross-issuer collisions (D8.2),
- identifier reassignment (D8.3),
- records being used as authentication (D8.11),
- privacy of immutable records (D8.12).

Residual: compromise of a trusted issuer. At the Observer grade, attested
identities are all that is available.

**Unresolved dependencies.**
- OQ-4 residual: key binding.
- OQ-6: how policy refers to identities.
- OQ-7: delegation.
- OQ-9: approver eligibility beyond D8.7.
- OQ-18: agents as verifiers.
- OQ-19: retention and redaction.
- OQ-21: recording the authentication boundary.

**Acceptance criteria.** D8 can be accepted, and OQ-5 marked resolved,
when the owner confirms:
- the concept distinctions (D8.1),
- the `(issuer, subject)` form (D8.2),
- the three kinds, including that models and organizations are not kinds
  (D8.4),
- the provenance categories and the no-upgrade rule (D8.5),
- the agent and model prohibitions (D8.7),
- the actor and principal attribution rules, including direct action and
  the delegation rule (D8.8),
- whether agents may ever be principals (D8.8),
- the provenance required for holder and approver checks (D8.9),
- the separation of recording from enforcement (D8.14).

### D9. CONTROL decision records (OQ-29)

**Status: Proposed.** Not accepted. OQ-29 remains open.

1. **Home.** A CONTROL decision is recorded in an independent **decision
   record**. Its format is specified as a distinct record form within the
   **Evidence Receipt** protocol (analysis option G). It is a TRUST › Audit
   record of a decision that CONTROL made. It is not a fifth core protocol,
   and it does not move any authority out of CONTROL.
2. **Independent and identifiable.** Each decision record:
   - is its own protocol document with its own content digest (D4.3),
   - is produced by CONTROL when the decision is made, whether or not the
     action executes.

   Denials, approver rejections and permits are all recorded. Whether the
   form is distinguished by its type identifier (D6.4) or by a core member
   is left to the Evidence Receipt specification.
3. **Bound to the exact proposal and its inputs.** A decision record:
   - MUST reference the Action IR proposal it decides by that proposal's
     content digest,
   - MUST reference by digest every protocol document it relied on, such as
     Capability Manifests and approvals,
   - MUST record the actor and principal attributions (D8.8) as the
     identity references CONTROL used, each with its D8.5 provenance. For
     a delegated action, it MUST also reference the validated delegation,
   - MUST state explicitly when no identity is available for either role,
     and MUST NOT substitute another identity,
   - MUST identify its producer.

   Identity requirements depend on the outcome, not on whether a record is
   kept:
   - A decision is recorded whatever the provenance of its identities.
     Denials, approver rejections and Observer-grade decisions MUST remain
     recordable when only trusted-attested or self-asserted identities, or
     none, are available.
   - A permit outcome MUST NOT rest on a self-asserted identity (D8.5). Its
     holder and approver checks follow D8.9.
   - A recorded provenance is the provenance CONTROL relied on at decision
     time. It MUST NOT be upgraded later (D8.5), and recording it never
     authenticates anyone (D8.11).

   How policy (OQ-6), risk (OQ-8) and approvals (OQ-9) are identified is
   left to those questions. A decision record MUST distinguish at least a
   permit outcome from a deny outcome. The complete outcome vocabulary is
   left to the Evidence Receipt specification and OQ-17.
4. **Not a credential.** Possessing or presenting a decision record, an
   Evidence Receipt, an identity recorded in either, or a copy of any of
   them never authorizes anything.
   - An enforcement point MUST accept authorization only from CONTROL's own
     evaluation of the exact proposal, received over a channel inside the
     controlled boundary.
   - That authorization MUST NOT be relayed through the requesting actor,
     model or host.
   - An enforcement point MUST NOT accept any decision record or receipt as
     permission to execute, whoever presents it (C17).
5. **Exact action, no substitution, no replay.**
   - Before executing, the enforcement point MUST confirm that the action
     it is about to execute has the same content digest as the proposal
     CONTROL evaluated.
   - Each execution attempt requires CONTROL authorization for that
     attempt. An earlier permit, or its record, MUST NOT authorize a new
     execution of an identical proposal.
   - Expiry and single-use semantics are OQ-9.
6. **No enforcement claims.** A decision record states what CONTROL
   decided. It MUST NOT state or imply that the decision was enforced.
   Enforcement is stated only in later Evidence Receipts, with the
   integration grade and enforcement boundary (C10, C16). At the Observer
   grade, decision records are still produced.
7. **Later references and immutability.**
   - Evidence Receipts reference decision records by content digest.
   - No execution-bound Action IR form is introduced, and Action IR remains
     a proposal (C6).
   - A decision record is immutable (D2.7). A revocation or change of
     decision is a new decision record that references its predecessor by
     digest.
   - A reference to an unavailable decision record leaves "authorized"
     unverifiable, never verified (C5).
8. **Authenticity.** Until a later ADR defines signing (D4.6, D4.7):
   - a decision record is trustworthy only within the trust boundary that
     produced it,
   - elsewhere it is a statement attributed to its producer, without
     established authenticity.
9. **Governance conditions.** Decision records remain within the Evidence
   Receipt protocol only while all of the following hold:
   - they are specified only in the Evidence Receipt specification,
   - they share its version line (D2.1), extension element and limits,
   - they have no namespace root of their own,
   - their only producer is the CONTROL plane of a runtime. Models, agents
     and hosts never produce them,
   - they define no conformance class separate from Evidence Receipt
     conformance.

   A change that would break any of these conditions requires a new ADR
   amending ADR 0001.

**Rationale.**
- Enforcement never needs to read a decision record, because authorization
  flows from CONTROL's own evaluation (D9.4). A record of a decision is
  therefore an audit artifact.
- ADR 0001 places "a durable record of decisions" in TRUST › Audit. The
  Evidence Receipt is the existing TRUST-plane protocol whose frozen
  purpose is a "machine-verifiable record linking actions, evidence, claims
  and verification".
- Specifying the record form there makes decision records independent,
  digest-referenced and third-party auditable, without a fifth protocol.

**Alternatives considered.**
- A, inside receipts only: no record of decisions about actions that never
  execute.
- B-int, an internal Audit format: not auditable by third parties.
- B-new, a separately versioned format: a fifth protocol in all but name.
- C, an execution-bound Action IR form: close to a credential.
- D, inside the Capability Manifest: conflicts with C7.
- E, a fifth core protocol: excluded.
- F, host logs only: insufficient.

See
[analysis § OQ-29](../protocols/shared-blockers-analysis.md#oq-29-control-decision-records).

**Security implications.**
- Credential misuse is addressed by D9.4.
- Proposal substitution and replay are addressed by D9.5.
- Overclaiming enforcement is addressed by D9.6.
- Missing records stay unverifiable (D9.7).
- Recording identities with their actual provenance keeps denials and
  Observer-grade decisions auditable, without upgrading weak identities
  (D9.3).
- Forgery cannot authorize anything, because records are never
  credentials. It can mislead an audit only until signing exists (D9.8,
  D3.7).

**Effect if accepted.**
- Action IR is no longer conditionally blocked by OQ-29.
- The Evidence Receipt specification must define the decision-record
  form.

**Unresolved dependencies.**
- OQ-6, OQ-8 and OQ-9: how decision inputs are identified.
- OQ-17: outcome vocabulary.
- OQ-20: audit integrity.
- OQ-21: recording the grade and boundary.
- OQ-4 residual: authenticity.

**Acceptance criteria.** D9 can be accepted, and OQ-29 marked resolved,
when the owner confirms:
- that a decision-record form within the Evidence Receipt protocol is
  within that protocol's frozen purpose, and is not a fifth core protocol
  (D9.1, D9.9),
- the identity and provenance recording rules (D9.3),
- the not-a-credential and exact-action rules (D9.4, D9.5),
- that no execution-bound Action IR form is introduced (D9.7).

If the owner does not accept the reading in the first point, the
alternative that complies with ADR 0001 is option B-int. It gives up
third-party auditability.

## Consequences

If all four decisions are accepted:

- Every protocol can carry a final type identifier, once `{root}` is
  recorded, and has stated limits.
- Every protocol can name actors without conflating identity and authority.
- Evidence Receipts and decision records have a defined relationship.
- Field-level specification still requires the per-protocol questions in
  the dependency map, and explicit authorization of that phase.
- Implementations need a streaming or bounded pre-parse check of the
  ceiling (D7.4), in addition to the components ADR 0002 already requires.

If only some decisions are accepted, the others remain open and continue to
block as shown in the dependency map.

## Preservation of ADR 0001 and ADR 0002

| Frozen or accepted element | Effect of this ADR |
|---|---|
| Four planes | Unchanged. Decision records are made by CONTROL and recorded in TRUST › Audit, both existing components. |
| Four core protocols | Unchanged. No protocol is added. D9 places a record form inside the Evidence Receipt protocol, under the conditions in D9.9. |
| I1 `MODEL != AUTHORITY` | Reinforced: D8.6, D8.7 and D9.4. |
| I2 `MODEL CLAIM != VERIFIED FACT` | Preserved: decision records state decisions, not verification outcomes. |
| I3 `MEMORY != POLICY` | Preserved: self-asserted identity never permits (D8.5). |
| I4 `TOOL OUTPUT != TRUSTED FACT` | Preserved: attested identity is labeled as such (D8.5). |
| I5 `HOST SUPPORT != ENFORCEMENT` | Preserved: decision records make no enforcement claims (D9.6). |
| I6 `NO EVIDENCE -> NO VERIFIED COMPLETION` | Preserved: unavailable records are unverifiable (D9.7). Unresolved references from exhausted budgets are never verified (D7). |
| I7 `NO CAPABILITY -> NO EFFECTFUL ACTION` | Preserved: identity is not a grant (D8.6). Capability holding is bound to the authenticated actor (D8.9). |
| I8 `PRIVILEGE EXPANSION -> EXTERNAL AUTHORIZATION` | Reinforced: agents cannot approve or grant (D8.7). Delegation never expands authority (D8.8). |
| I9 `HIGH-RISK ACTION -> POLICY / APPROVAL` | Preserved: approvals are recorded by digest (D9.3). Their semantics remain OQ-9. |
| I10 `FAILED TRANSACTION -> ROLLBACK OR COMPENSATION` | Unaffected. |
| Integration grades | Preserved: D8.5, D8.14, D9.6. |
| ADR 0002 D1 to D5 | Unchanged. D6 fills the namespace that D2.2 and D3.2 left open. D7 supplies the limits that D1.4 requires, and changes limits only within D2's version rules. D8 and D9 use D4 digests and D5 adapter boundaries without changing them. |

No conflict with ADR 0001 or ADR 0002 was found. The one interpretive
point, whether a decision-record form fits within the Evidence Receipt's
frozen purpose, is surfaced for the owner in D9's acceptance criteria.

## Owner decisions required

1. **D6.1:** choose namespace option A (proposed) or C. Choose and register
   the domain, and name the registrant. Under C, also choose the minting
   date and record the evidence of control (D6.8). Record `{root}`.
   Option B (w3id.org) is available only through a separate governance
   decision and a new ADR that changes the control requirement.
2. **D6.2 to D6.8:** confirm or amend the never-dereference rule, canonical
   spelling, template, minting rule, safeguards, placeholder and the
   conditions for option C.
3. **D7.3:** confirm or amend each ceiling value.
4. **D7.5, D7.6:** confirm the version rule for limits and the
   local-refusal rule.
5. **D8.2, D8.4:** confirm the `(issuer, subject)` form and the three
   kinds.
6. **D8.7:** confirm the prohibitions on agents and model descriptors.
7. **D8.8:** confirm that agents are never principals, or choose the
   alternative in the analysis.
8. **D8.9:** choose the provenance required for capability-holder checks:
   authenticated only; authenticated by default with explicit per-attester
   allowance (proposed); or any trusted-attested identity. Confirm that
   approvers must be authenticated.
9. **D9.1, D9.9:** confirm that a decision-record form within the Evidence
   Receipt protocol is within ADR 0001's frozen purpose for that protocol,
   or reject it in favor of option B-int.
10. **Per decision:** accept, amend or reject D6, D7, D8 and D9
   independently.
