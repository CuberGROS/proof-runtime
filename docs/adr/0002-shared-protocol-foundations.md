# ADR 0002: Shared Protocol Foundations

- **Status:** Proposed (pending review by the repository owner)
- **Date:** 2026-10-02
- **Phase:** 1A — protocol foundations
- **Proposes resolutions for:** OQ-1, OQ-2, OQ-3, OQ-4 (partially) and OQ-23
  from [ADR 0001](0001-architecture-freeze.md#open-questions)
- **Does not change:** ADR 0001, the four planes, the four core protocols,
  the ten invariants or the three integration grades

This ADR stays **Proposed** until the repository owner explicitly accepts
it. Until then, OQ-1, OQ-2, OQ-3, OQ-4 and OQ-23 remain open, and field-level
specification of any protocol stays blocked.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in the
Decision section are to be interpreted as described in BCP 14 (RFC 2119 and
RFC 8174) when, and only when, they appear in all capitals.

## Context

ADR 0001 froze four core protocols (Task Capsule, Action IR, Capability
Manifest and Evidence Receipt) and required each to be model-neutral,
host-neutral, versioned and extensible. It deliberately did not choose an
encoding, a versioning scheme, an extension model or an integrity mechanism.
Those choices are shared by all four protocols. Making them separately for
each protocol would produce four incompatible conventions. So they are
decided once, here, before any protocol is specified.

The detailed analysis behind each decision is in
[docs/protocols/foundations-analysis.md](../protocols/foundations-analysis.md):
requirements, alternatives, interoperability, security consequences,
migration and residual risks. The cross-protocol constraints these
decisions must satisfy are in
[docs/protocols/design-constraints.md](../protocols/design-constraints.md).
The standards evaluation is in
[docs/protocols/standards-matrix.md](../protocols/standards-matrix.md).

## Decision

This ADR decides only shared, cross-protocol conventions. It defines no
protocol fields. Where it names an information element that every document
must carry (for example, "a protocol version"), the element's field name,
position and exact syntax are left to the protocol specifications.

### D1. Encoding and schema (OQ-1)

1. **Interchange encoding.** Every protocol document is a JSON text
   (RFC 8259), encoded in UTF-8, whose top-level value is an object.
2. **Proof Runtime JSON profile.** Documents MUST also conform to I-JSON
   (RFC 7493). In addition:
   - Producers MUST NOT emit duplicate member names. Consumers MUST reject
     documents that contain them, rather than applying "last one wins".
   - Producers MUST NOT emit a byte order mark. Consumers MUST reject
     documents that start with one.
   - **Numbers.** Core members (members defined by a protocol
     specification, not by an extension) MUST NOT use non-integer JSON
     numbers. Integer values MUST lie in the range
     −(2^53 − 1) to 2^53 − 1. Values outside that range, decimals and
     quantities that need exact precision are represented as strings in a
     format the protocol specification defines.
   - **Absence.** A core member that has no value is omitted. `null` is not
     used to mean "absent" in core members.
   - **Timestamps** use RFC 3339 with an explicit `Z` (UTC) offset. A
     timestamp is the producer's claim about time, not trusted time (see
     OQ-28).
   - **Binary content** is not embedded in core members. It is referenced
     by digest (D4). Where a protocol specification must embed bytes, it
     uses unpadded base64url (RFC 4648 §5).
   - **Identifiers** that are compared by machines (type identifiers,
     extension identifiers, member names) are restricted to ASCII and
     compared as exact, case-sensitive strings, with no normalization.
3. **Schema language.** Each protocol specification publishes a JSON Schema
   (draft 2020-12) for structural validation.
   - The prose specification is normative. The schema is a normative
     **minimum**: a document that fails schema validation is invalid, but
     passing validation does not make a document valid. Semantic rules that
     the schema cannot express remain in the prose.
   - Schemas MUST be resolvable offline, by `$id`, from schemas bundled with
     the implementation. Validators MUST NOT fetch `$ref` targets from the
     network while validating documents.
   - Schemas MUST NOT rely on the `format` keyword for any security-relevant
     check, because in 2020-12 `format` is an annotation by default and is
     not asserted unless a validator opts in.
4. **Resource limits.** Every protocol specification MUST define maximum
   document size and maximum nesting depth. Consumers MUST reject documents
   that exceed them before any further processing. The actual limits are
   open (OQ-27).
5. **Other encodings.** CBOR and other binary encodings are not part of
   v0.1. Adding one requires a new ADR that defines a lossless mapping from
   the JSON data model and its own deterministic encoding.

### D2. Versioning and compatibility (OQ-2)

1. **Independent versions.** Each of the four protocols is versioned
   independently. There is no single "Proof Runtime version" on the wire.
2. **Self-description.** Every protocol document MUST carry two pieces of
   information:
   - a **document type identifier**: an absolute URI that names the protocol
     and its major version. Its namespace is open (OQ-26).
   - a **protocol version** in the form `MAJOR.MINOR`.

   Patch-level (editorial) revisions of a specification do not appear in
   documents.
3. **Major version.** A major version changes when any change could cause a
   consumer of the previous major version to misinterpret a document.
   A consumer MUST reject a document whose major version it does not
   implement. It MUST NOT attempt best-effort interpretation.
4. **Minor version.** A minor version MAY only add optional core members, or
   new values that the specification marks as ignorable. A minor revision
   MUST NOT change the meaning of an existing member.
   - A consumer that implements an earlier minor version of the same major
     version MAY process the document. It ignores core members it does not
     recognize, subject to D2.5 and D3.
5. **Monotonic rule.** A change whose omission or non-recognition could
   cause a consumer to grant more authority, widen a capability, accept
   weaker evidence, or report a stronger verification outcome MUST NOT be
   introduced as an ignorable minor addition. It MUST be introduced either
   as a new major version or as a critical extension (D3.4).
6. **Pre-1.0.** Protocol versions `0.x` carry no compatibility promise
   between minor versions. Compatibility rules D2.3 to D2.5 bind from
   version `1.0` of each protocol.
7. **Immutability and migration.** A document identified by a content
   digest (D4) is never edited in place.
   - Migrating a document to a new version produces a new document. The
     new document references the original by digest, and the original is
     retained.
   - A migrated document is attributed to whoever performed the migration,
     not to the producer of the original.
8. **Unsupported versions** are reported as explicit, recorded errors. A
   consumer MUST NOT silently downgrade a document, or guess at a newer one.

### D3. Extension model (OQ-3)

1. **Location.** Extension data is carried only in a single designated
   extensions element of each document. It is keyed by extension
   identifier. Extensions MUST NOT add members alongside core members, and
   MUST NOT redefine the meaning of core members.
2. **Identifiers.** An extension identifier is an absolute URI (RFC 3986)
   under the control of the extension's author.
   - The URI includes the extension's major version.
   - Identifiers are compared as exact strings (D1.2).
   - The core namespace (OQ-26) is reserved for protocol specifications.
3. **Non-critical extensions.** A consumer that does not understand a
   non-critical extension ignores it for every decision it makes. It
   preserves the extension byte-for-byte whenever it stores or relays the
   document.
4. **Critical extensions.** Each document MUST carry a list of the extension
   identifiers that are critical for that document (the list may be empty).
   - A consumer that does not understand every listed critical extension
     MUST reject the document for any authorization, verification or
     execution decision. It MUST NOT process the document partially.
   - Listing an identifier that is not present in the extensions element
     makes the document invalid.
5. **Monotonic rule for extensions.** Evaluating a document with all of its
   non-critical extensions removed MUST yield the same or a stricter
   outcome. Concretely, it must not lead to more authority, a wider
   capability, or a stronger verification outcome.
   - Any extension that can relax an authorization, widen a capability, or
     strengthen a verification outcome MUST be marked critical.
   - Extensions are expected to restrict or annotate; relaxing requires
     criticality.
6. **Stripping attacks.** Removing an extension or a critical marker changes
   the document's content digest (D4).
   - Before signatures exist, this detects tampering only when a consumer
     already holds the expected digest from an independent source.
   - Protection against an intermediary that strips criticality and
     re-presents the document requires authenticity (D4.5). This is a
     stated residual risk, not a solved one.
7. **Industry semantics** (software engineering, finance, healthcare and so
   on) live only in extensions, never in core members.
8. **Registry.** v0.1 has no central extension registry. Collision
   resistance comes from URI ownership.

### D4. Canonicalization and integrity (OQ-4, partial)

1. **Four distinct properties.** Proof Runtime documents and implementations
   MUST keep these apart:
   - **Integrity**: these bytes have not changed since a digest was
     computed.
   - **Authenticity**: a particular key holder endorsed these bytes. This
     requires signatures and a binding from keys to identities (OQ-5).
   - **Authorization**: the endorsing party was permitted to make this
     statement or decision. This is a CONTROL-plane question.
   - **Correctness**: the content is true. This is a TRUST-plane question,
     answered only by verification against evidence (I2, I6).

   No cryptographic mechanism in this ADR establishes authorization or
   correctness. A valid digest or signature over an Evidence Receipt does
   not show that execution happened as described.
2. **Canonical form.** The canonical form of a protocol document is its
   JSON Canonicalization Scheme (JCS, RFC 8785) serialization. The D1
   profile, in particular integer-only core numbers and ASCII identifiers,
   exists partly to keep JCS output identical across languages.
3. **Content digest.** A document's content digest is computed over its
   canonical form. Opaque artifacts (logs, files, binaries) are digested
   over their exact bytes, with no canonicalization.
   - A document does not contain its own digest. Documents reference other
     documents and artifacts by digest.
4. **Digest representation and agility.** Digests are represented as a
   mapping from algorithm name to lowercase hex value. This is compatible
   with the in-toto `DigestSet`.
   - Implementations MUST support `sha256`.
   - Consumers MUST accept only algorithms they consider secure, and MUST
     treat a reference with no acceptable algorithm as unverifiable, not as
     matching.
   - Adding or retiring algorithms does not require a protocol major
     version.
5. **Future signatures.** Signing is not implemented in v0.1, and no signing
   infrastructure exists. When signing is introduced:
   - The preferred envelope is DSSE. It signs exact payload bytes together
     with a payload type, using its pre-authentication encoding.
   - Producers SHOULD place the canonical form (D4.2) in the payload, so
     that the signed bytes and the content digest refer to the same bytes.
   - Signatures are carried outside the document they sign, so signing never
     changes a document's content digest.
   - Which parties sign which documents, key management, rotation,
     revocation and trust roots remain open (OQ-4 residual, OQ-5).
6. **No laundering of provenance.** A signature by the runtime over a record
   that contains host-attested or observed evidence attests only that the
   runtime recorded it. It MUST NOT be presented as converting that evidence
   into runtime-enforced fact (I4, I5).

### D5. Relationship to existing standards (OQ-23)

Proof Runtime **reuses** a small set of established specifications
normatively, **interoperates** with adjacent agent and observability
protocols at defined boundaries, and **aligns** conceptually with
provenance and attestation models without adopting their full data models
in v0.1. The per-standard classification is in
[docs/protocols/standards-matrix.md](../protocols/standards-matrix.md).
The binding rules are:

1. **Normative reuse in v0.1:**
   - RFC 8259 (JSON), RFC 7493 (I-JSON), RFC 8785 (JCS),
   - JSON Schema 2020-12,
   - RFC 3339 (timestamps), RFC 4648 (base64url), RFC 3986 (URIs),
   - DSSE as the designated future signing envelope,
   - the in-toto `DigestSet` shape.
2. **Interoperability, not dependency.** MCP, A2A, CloudEvents,
   OpenTelemetry, in-toto attestations, SLSA, OIDC and SPIFFE are treated as
   external protocols that adapters may map to or from. No core protocol
   requires any of them, and no core member's meaning depends on them.
3. **No compliance claims.** Proof Runtime makes no claim of conformance,
   compatibility or certification with any external standard until a
   mapping is specified, implemented and tested against that standard's own
   conformance material.
4. **External signals are not authority.** Metadata from external protocols
   is treated as host-attested or tool-provided input, never as a CONTROL
   decision (I1, I4, I5). Examples are MCP tool annotations, A2A Agent Card
   capability declarations and telemetry attributes.

## Consequences

- Protocol specifications can begin once this ADR is accepted, subject to
  the protocol-specific open questions listed in
  [docs/protocols/dependency-map.md](../protocols/dependency-map.md).
- Every implementation needs:
  - a strict JSON parser that detects duplicate names,
  - a JCS implementation,
  - SHA-256,
  - an offline JSON Schema 2020-12 validator.

  JCS implementations exist in several languages (see the standards matrix),
  but cross-implementation agreement must be confirmed by shared test
  vectors before it is relied on.
- Documents become immutable, digest-addressed records. Corrections,
  migrations and redactions create new documents that reference earlier
  ones.
- Fail-closed behavior on unknown majors and unknown critical extensions
  means a newer producer can make an older consumer refuse a document. This
  is intended: refusal is preferred to silent weakening.
- Extension authors bear responsibility for marking criticality correctly.
  An extension that relaxes controls without being marked critical violates
  D3.5, and conformant consumers must not honor the relaxation.

## Preservation of ADR 0001

| Frozen element                                       | Effect of this ADR                                                                                                                  |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Four planes                                          | Unchanged. No plane component is added or redefined.                                                                                |
| Four core protocols                                  | Unchanged. No protocol is added; no fields are defined.                                                                             |
| I1 `MODEL != AUTHORITY`                              | Preserved: an Action IR document carries no authority; external metadata is never a CONTROL decision (D5.4).                        |
| I2 `MODEL CLAIM != VERIFIED FACT`                    | Preserved: integrity and signatures are separated from correctness (D4.1).                                                         |
| I3 `MEMORY != POLICY`                                | Preserved: nothing in a document's encoding, version or extensions grants authority by itself (D3.5).                              |
| I4 `TOOL OUTPUT != TRUSTED FACT`                     | Preserved: hashing and signing do not upgrade provenance (D4.6, D5.4).                                                             |
| I5 `HOST SUPPORT != ENFORCEMENT`                     | Preserved: external capability declarations are host-attested input (D5.4); runtime signatures do not launder provenance (D4.6). |
| I6 `NO EVIDENCE -> NO VERIFIED COMPLETION`           | Preserved: unknown or unacceptable digests are unverifiable, never matching (D4.4); critical extensions fail closed (D3.4).        |
| I7 `NO CAPABILITY -> NO EFFECTFUL ACTION`            | Preserved: an extension that relaxes controls must be critical, so it is refused rather than honored where not understood (D3.5).  |
| I8 `PRIVILEGE EXPANSION -> EXTERNAL AUTHORIZATION`   | Preserved: the monotonic rules (D2.5, D3.5) prevent widening by version or extension skew.                                         |
| I9 `HIGH-RISK ACTION -> POLICY / APPROVAL`           | Preserved: digest-addressed documents make it possible to bind an approval to exact bytes; whether approvals must do so is OQ-9.   |
| I10 `FAILED TRANSACTION -> ROLLBACK OR COMPENSATION` | Unaffected; transaction semantics remain OQ-11.                                                                                     |
| Integration grades                                   | Preserved: no mechanism here creates an enforcement claim beyond the boundary the runtime controls.                                |

No conflict with ADR 0001 was found. One gap was found and is recorded as
OQ-29 below. Separating authorization from proposals (I1) requires a home
for CONTROL decision records, which none of the four protocols is yet
assigned to carry. This ADR records the gap and does not amend ADR 0001.

## New open questions

These questions arise from this ADR and continue the numbering in ADR 0001.
They do not modify ADR 0001.

- **OQ-26 Identifier namespace.** Which URI authority will host Proof Runtime
  document type identifiers and the reserved core extension namespace? It
  must be durable and under the project's control. Resolving this needs an
  owner decision about a domain or other namespace.
- **OQ-27 Resource limits.** What maximum document size, nesting depth,
  string length and array length does each protocol permit?
- **OQ-28 Trusted time.** Do any records require time from a trusted source
  (for example, RFC 3161 timestamping or a transparency log), rather than
  producer-asserted timestamps? If so, which ones?
- **OQ-29 Home of CONTROL decision records.** Authorization must be a
  CONTROL-plane decision recorded separately from the Action IR proposal it
  concerns (design constraint C6). Which existing artifact carries these
  decision records? For example, they could be recorded as evidence within
  Evidence Receipts, or kept as Audit records outside the four protocol
  documents. This ADR does not decide the question, and it must not be
  resolved by adding a fifth core protocol without a new ADR that amends
  ADR 0001.

The residual part of **OQ-4** stays open: which parties sign which documents,
and how keys are managed, rotated and revoked. It depends on OQ-5.

## Verification status of cited standards

The research for this ADR was done with restricted network access. Some
primary sources were read directly; others could not be reached and are
cited from prior knowledge. The standards matrix records the status of each
citation. Claims marked "not verified in this review" must be checked
against the primary source before this ADR is accepted.
