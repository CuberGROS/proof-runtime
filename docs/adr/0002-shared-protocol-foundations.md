# ADR 0002: Shared Protocol Foundations

- **Status:** Accepted
- **Date:** 2026-10-02
- **Accepted:** 2026-10-02, by the repository owner and maintainer
  (@CuberGROS)
- **Phase:** 1A — protocol foundations
- **Resolves:** OQ-1, OQ-2, OQ-3 and OQ-23, and OQ-4 except its residual,
  from [ADR 0001](0001-architecture-freeze.md#open-questions)
- **Does not change:** ADR 0001, the four planes, the four core protocols,
  the ten invariants or the three integration grades

This ADR was proposed and reviewed in pull request #2. The repository owner
and maintainer explicitly accepted it, and that acceptance is recorded in
this change.

- **Resolved by this ADR:** OQ-1, OQ-2, OQ-3 and OQ-23. OQ-4 is resolved
  except its residual: which signing envelope, which parties sign, and key
  management.
- **Still open:** the OQ-4 residual, and OQ-26 to OQ-29, which this ADR
  raises.
- **Record-keeping:** ADR 0001's open-question list is unchanged; this ADR
  records the resolutions.
- **Changes:** changing any decision below requires a new ADR.
- **Post-acceptance clarifications.** After acceptance, review in pull
  request #2 led to clarifications, intended to leave every accepted
  decision unchanged:
  - the scope of D1 to D5 (Scope and duration),
  - schema validation for newer minor versions (D1.3, D2.4),
  - the canonical protocol version syntax and comparison (D2.2),
  - SHA-256 in the normative reuse list (D5.1).

  The owner's acceptance above predates them. Owner confirmation of these
  clarifications is not recorded in this ADR.

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

Supporting documents:

- [Analysis](../protocols/foundations-analysis.md): requirements,
  alternatives, interoperability, security consequences, migration and
  residual risks behind each decision.
- [Design constraints](../protocols/design-constraints.md): the
  cross-protocol constraints these decisions must satisfy.
- [Standards matrix](../protocols/standards-matrix.md): the standards
  evaluation, with the verification status of each citation.
- [Conformance cases](../protocols/conformance-cases.md): test
  cases for the extension, version and integrity rules.

## Decision

This ADR decides only shared, cross-protocol conventions. It defines no
protocol fields. Where it names an information element that every document
must carry (for example, "a protocol version"), the element's field name,
position and exact syntax are left to the protocol specifications, except
that D2.2 fixes the syntax of the protocol version value.

### Scope and duration

- **Every version of every core protocol.** D1 to D5 govern every version
  of all four core protocols (Task Capsule, Action IR, Capability Manifest
  and Evidence Receipt), before and after `1.0`. They continue to govern
  until a later ADR explicitly supersedes the applicable rule; a new
  protocol version alone never does.
- **Not a wire version.** "Phase 1A" and the name "v0.1 foundation" refer
  to the project milestone in which this ADR was written. They are not a
  protocol version and do not limit any rule to protocol version `0.1` or
  to `0.x` versions. This ADR introduces no global protocol version;
  protocols remain independently versioned (D2.1).
- **Reserved items.** Where this ADR marks an item as reserved (D4.7), it
  imposes no requirement until a later ADR adopts it.

### Terms used in this decision

- **Security-relevant decision.** Any decision about authorization,
  capability scope, whether or how an action executes, or a verification
  outcome.
- **Decision-neutral.** Content is decision-neutral if no conformant
  consumer uses it as input to a security-relevant decision. Its presence
  or absence therefore cannot change such a decision.

### D1. Encoding and schema (OQ-1)

1. **Interchange encoding.** Every protocol document is a JSON text
   (RFC 8259), encoded in UTF-8, whose top-level value is an object.
2. **Proof Runtime JSON profile.** Documents MUST also conform to I-JSON
   (RFC 7493). In addition:
   - Producers MUST NOT emit duplicate member names. Consumers MUST reject
     documents that contain them, rather than applying "last one wins".
   - Producers MUST NOT emit a byte order mark. Consumers MUST reject
     documents that start with one.
   - **Numbers: one strict integer profile for the whole document.**
     Every JSON number token anywhere in a document, core or
     extension at any nesting depth, MUST satisfy all of the following:
     - **Integer spelling only.** The token consists of an optional minus
       sign followed by `0` or by a nonzero digit and further digits. It
       contains no fraction part and no exponent part. Integral values
       written as `1.0`, `1e0` or `9.007199254740993e15` are therefore
       invalid.
     - **No negative zero.** The token `-0` is invalid.
     - **Safe range.** The value lies in the range −(2^53 − 1) to
       2^53 − 1 (−9007199254740991 to 9007199254740991).
     - **Lexical validation first.** Consumers MUST check these rules on
       the token text itself, before any conversion to binary64 or any
       other lossy representation. A parser that rounds first cannot
       detect an out-of-range value such as `9007199254740993`.

     Consumers MUST reject a document containing any number token that
     violates these rules. Fractions, monetary amounts, integers outside
     the safe range and any other quantity that needs exact precision MUST
     be encoded as strings, in a format defined by the specification that
     owns the member.

     This profile is intentionally stricter than JCS (RFC 8785), which
     accepts any number representable as an IEEE 754 double. Every
     conforming number therefore has exactly one textual form and one exact
     value in every language, and JCS serializes it as its plain decimal
     integer spelling.
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
   - Consumers MUST reject a document that fails validation against the
     normative schema of the protocol version the document declares (D2.2).
     This applies equally when a consumer processes a newer minor version
     under D2.4.
   - A consumer that cannot resolve that schema offline MUST reject the
     document. It MUST NOT substitute the schema of another version, skip
     validation, or fetch the schema from the network.
   - Schemas MUST be resolvable offline, by `$id`, from schemas bundled with
     the implementation. Validators MUST NOT fetch `$ref` targets from the
     network while validating documents.
   - Schemas MUST NOT rely on the `format` keyword for any security-relevant
     check. In 2020-12, `format` is collected as an annotation, and
     assertion is disabled by default (JSON Schema Validation 2020-12
     §7.2.1).
4. **Resource limits.** Every protocol specification MUST define maximum
   document size and maximum nesting depth. Consumers MUST reject documents
   that exceed them before any further processing. The actual limits are
   open (OQ-27).
5. **Other encodings.** CBOR and other binary encodings are not part of
   the core protocols. Adding one requires a new ADR that defines a
   lossless mapping from the JSON data model and its own deterministic
   encoding.

### D2. Versioning and compatibility (OQ-2)

1. **Independent versions.** Each of the four protocols is versioned
   independently. There is no single "Proof Runtime version" on the wire.
2. **Self-description and consistency.** Every protocol document MUST carry
   two pieces of information:
   - a **document type identifier**: an absolute URI that names the protocol
     and its major version. Its namespace is open (OQ-26).
   - a **protocol version** in the form `MAJOR.MINOR`.

   The major version encoded in the type identifier MUST equal the `MAJOR`
   of the declared protocol version. A document in which they differ is
   invalid, and consumers MUST reject it. Patch-level (editorial) revisions
   of a specification do not appear in documents.

   **Canonical version syntax.** A protocol version has exactly one
   spelling:

   ```abnf
   version   = component "." component   ; MAJOR "." MINOR
   component = "0" / ( %x31-39 0*8DIGIT ) ; DIGIT = %x30-39 (ASCII only)
   ```

   - Each component is ASCII decimal: `0`, or a digit `1` to `9` followed
     by up to eight further digits. Its value is therefore in the range 0
     to 999999999.
   - Leading zeros, signs, whitespace, additional components, non-ASCII
     digits and any other notation (for example hexadecimal or exponent
     forms) are not permitted. `01.2`, `1.02`, `+1.2`, `1.2.0`, `1`,
     `1.`, ` 1.2` and `0x1.2` are all invalid.
   - Producers MUST emit only this form. Consumers MUST validate the
     version lexically and MUST reject a document whose protocol version
     does not match it. They MUST NOT normalize a non-canonical spelling.
   - Because D1.2 forbids fraction numbers, a protocol version carried in
     JSON is a string, not a number.
   - The major version in the type identifier uses the same component
     syntax; its exact position in the URI is left to OQ-26 and the
     protocol specifications.
   - **Comparison.** Two versions are equal only if their MAJOR values are
     equal and their MINOR values are equal; for canonical spellings this is
     exact string equality. Versions are ordered by MAJOR value, then by
     MINOR value, each compared as a non-negative integer. `1.10` is
     therefore newer than `1.9`. Versions MUST NOT be compared as strings
     for ordering, or as decimal fractions (under which `1.1` and `1.10`
     would coincide).
3. **Major version.** A major version changes when any change could cause a
   consumer of the previous major version to misinterpret a document.
   A consumer MUST reject a document whose major version it does not
   implement. It MUST NOT attempt best-effort interpretation.
4. **Minor version (from 1.0).** For protocol versions `1.0` and later, a
   minor version MAY only add:
   - optional core members, or
   - new values to an existing member, but only if the specification
     version that first defined the member declared its value space
     **open** and specified how consumers handle unknown values.

   Every such addition MUST be decision-neutral (D2.5). An unknown value of
   an open member is handled exactly as that original definition
   specifies. That handling must itself be either decision-neutral or a
   rejection. Adding values to a member whose value space was not declared
   open requires a new major version. A minor revision MUST NOT change the
   meaning of an existing member or value.
   - A consumer that implements `M.n` MAY process a document declaring
     `M.m` with `m > n`. Doing so is optional; rejecting the document also
     conforms.
   - If it processes such a document, it MUST first validate it against
     the normative schema of `M.m`, resolved offline (D1.3). If that schema
     is not available to it, it MUST reject the document. Validation
     against the schema of `M.n` is not a substitute.
   - After validation, it ignores core members it does not recognize and
     handles unknown values of open members as originally specified. This
     is safe only because of D2.5.
   - A consumer MAY process documents declaring any `M.m` with `m ≤ n`.
5. **Security-relevant changes are never ignorable.** Any change that
   affects a security-relevant decision MUST be introduced either as a new
   major version or as a critical extension (D3.4). This applies whether
   the change would make decisions stricter or more permissive. Ignoring a
   new restriction is as much a divergence from the producer's intent as
   ignoring a new permission.
6. **Pre-1.0 versions: exact match only.** Protocol versions `0.x` carry no
   compatibility between minor versions, and none may be declared.
   - A consumer MUST reject a `0.x` document whose exact `MAJOR.MINOR` it
     has not explicitly implemented. A consumer may implement several
     `0.x` versions, but each one must be implemented individually.
   - A protocol specification MUST NOT declare two `0.x` minor versions
     compatible. No such compatibility is implied by version numbering.
   - D2.4 applies only from `1.0`. Before `1.0`, there is therefore no path
     by which a consumer accepts a document version it does not implement,
     and so no path for silently ignoring content added in a newer `0.x`
     minor version.
7. **Immutability and migration.** A document identified by a content
   digest (D4) is never edited in place.
   - **Immutable while retained, not retained forever.** Immutability
     governs a document's content for as long as it is retained. Whether a
     document is retained, for how long, and whether it may be deleted or
     replaced by a tombstone is decided by the retention and privacy policy
     (OQ-19), not by this ADR. Deletion removes a document; it never edits
     one.
   - A correction, migration (for example, to a new version) or redaction
     produces a new document. The new document MUST reference its
     predecessor by the predecessor's content digest, and MUST NOT use that
     digest as its own (D4.5). The predecessor is retained or deleted
     according to that policy.
   - **Unavailable is never verified.** A reference whose target has been
     deleted, tombstoned or is otherwise unavailable is unverifiable. Such
     historical evidence MUST NOT be presented as verified.
   - A migrated document is attributed to whoever performed the migration,
     not to the producer of the original.
   - D4.5 governs how digests behave across migration.
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
   - **Stable semantics per identifier.** Once an extension identifier is
     published, its semantics for security-relevant decisions are fixed.
     Any change that could alter a security-relevant decision made by a
     consumer of the earlier definition MUST be published under a new
     identifier. This includes adding a restricting or permitting member,
     or a new value with decision effect. Under the same identifier, an
     extension may change only editorially, or add decision-neutral content
     whose unknown-content handling was specified by its first definition.
     A consumer that "understands" an identifier (D3.4) therefore
     understands every security-relevant meaning that identifier can carry.
3. **Non-critical extensions are decision-neutral.** A non-critical
   extension MUST be decision-neutral. Every consumer, including one that
   understands the extension, MUST NOT use it as input to a
   security-relevant decision. Typical non-critical content includes
   display hints, correlation identifiers and descriptive domain metadata.
   - A consumer that does not understand a non-critical extension ignores
     it.
   - Its handling when the document is stored, relayed or transformed is
     governed by D4.5.
4. **Critical extensions.** Each document MUST carry a list of the extension
   identifiers that are critical for that document (the list may be empty).
   - A consumer that does not understand every listed critical extension,
     at the listed major version, MUST reject the document for every
     security-relevant decision. It MUST NOT process the document
     partially.
   - Listing an identifier that is not present in the extensions element
     makes the document invalid.
   - Identifiers in the critical list MUST be unique and ASCII (D1.2). A
     list that contains the same identifier more than once makes the
     document invalid.
5. **What must be critical.** An extension whose semantics can affect a
   security-relevant decision MUST be marked critical in every document
   that carries it. This applies to restrictive extensions (for example,
   "deny this action outside working hours") exactly as it does to
   permissive ones (for example, "also allow this action"). An unknown
   extension can therefore never silently weaken or strengthen a decision.
   - It is either decision-neutral and ignored (D3.3), or critical and
     fails closed (D3.4).
6. **Extension specifications declare their class.** Every extension
   specification MUST state whether the extension is critical (it affects
   security-relevant decisions) or non-critical (decision-neutral).
   - A consumer that understands an extension specified as critical, and
     finds it in a document that does not list it as critical, MUST reject
     the document as invalid.
   - A consumer that does not understand the extension cannot detect a
     missing critical marker. This producer error is a stated residual
     risk, and for unaware consumers it is **unmitigated**.
     - Producer-side conformance testing reduces how often it occurs.
     - Future signatures (D4.7) would attribute the error to its producer,
       but they would not let an unaware consumer reject the document. They
       are therefore not a mitigation.
     - A mechanism that would let unaware consumers detect the omission,
       such as authenticated extension-classification metadata, is not
       defined by this ADR.
7. **Stripping attacks.** Removing an extension or a critical marker changes
   the document's content digest (D4).
   - Before signatures exist, this detects tampering only when a consumer
     already holds the expected digest from an independent source.
   - Protection against an intermediary that strips criticality and
     re-presents the document requires authenticity (D4.7). This is a
     stated residual risk, not a solved one.
8. **Industry semantics** (software engineering, finance, healthcare and so
   on) live only in extensions, never in core members.
9. **Registry.** There is no central extension registry. Collision
   resistance comes from URI ownership.

### D4. Canonicalization and integrity (OQ-4, partial)

D4.1 to D4.6 are **normative**. D4.7 is **reserved**: it records a
direction for future work and imposes no requirement until a later ADR
adopts it.

1. **Four distinct properties.** Proof Runtime documents and implementations
   MUST keep these apart:
   - **Integrity**: the content has not changed since a digest was
     computed. It takes two forms:
     - For **protocol documents**, it is *canonical-content integrity*. The
       digest is over the JCS canonical form (D4.2, D4.3), so it shows that
       the JSON data model is unchanged. It does **not** show that the
       received bytes are unchanged. Reordered members, different whitespace
       or a different JSON string spelling of the same parsed content
       (`"\u0061"` versus `"a"`) produce the same digest.
     - For **opaque artifacts**, it is *byte integrity* over the exact bytes
       (D4.3).
   - **Authenticity**: a particular key holder endorsed these bytes. This
     requires signatures and a binding from keys to identities (OQ-5).
   - **Authorization**: the endorsing party was permitted to make this
     statement or decision. This is a CONTROL-plane question.
   - **Correctness**: the content is true. This is a TRUST-plane question,
     answered only by verification against evidence (I2, I6).

   No cryptographic mechanism in this ADR establishes authorization or
   correctness. A valid digest over an Evidence Receipt does not show that
   execution happened as described.
2. **Canonical form.** The canonical form of a protocol document is its
   JSON Canonicalization Scheme (JCS, RFC 8785) serialization. The D1
   profile, in particular its integer-only numbers and ASCII identifiers,
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
   - **`sha256` in every reference.** Every digest reference in a
     document MUST include a `sha256` entry. A reference without one is
     invalid.
   - Producers MUST compute every entry in a digest set over the same
     content.
   - Consumers MUST accept only algorithms they consider secure. `sha256`
     is always accepted while it is the mandatory common anchor.
   - **Matching: a common anchor, plus all-of-accepted.** Candidate content
     matches a digest set only if both of these hold:
     1. the `sha256` entry matches. Every consumer MUST verify it; it is
        the common validation anchor; and
     2. every other entry, for an algorithm the consumer accepts, also
        matches.

     If any verified entry does not match, the set does not match the
     candidate. Entries for algorithms the consumer does not accept are not
     verified by that consumer.
     - Every conformant consumer verifies the same `sha256` entry. Two
       conformant consumers can therefore never resolve one reference to
       different content. If a producer violates the same-content rule, a
       consumer that verifies an extra failing entry rejects, while another
       may accept the content identified by `sha256`. That divergence fails
       closed; it never yields different content.
     - This is stricter than the in-toto `DigestSet` guidance, under which
       sets "SHOULD be considered matching if ANY acceptable field matches".
   - Adding algorithms does not require a protocol major version.
     Retiring or replacing `sha256` as the mandatory common anchor
     requires a new ADR.
5. **Stored, relayed and transformed documents.**
   - **Recompute, never trust a stated digest.** A digest is valid for a
     document only if it is recomputed from the document actually held.
     A digest value stored or transmitted alongside a document MUST NOT be
     accepted for that document without recomputation.
   - **Stored or relayed unchanged.** A party that stores or relays a
     document SHOULD retain and forward its original bytes. If it parses
     and re-serializes the document instead, it MUST preserve the JSON data
     model losslessly: every member, including unknown extension data, with
     identical values. The content digest of the re-serialized document
     then equals the original's. A party that cannot guarantee lossless
     preservation, for example because its parser cannot represent a value
     exactly, MUST forward the original bytes, or treat its output as a
     transformation.
   - **Byte equality is not promised.** Neither of the above promises
     byte-for-byte equality after parsing and re-serialization. Only the
     content digest over the canonical form is preserved.
   - **Transformed.** Any change to the data model produces a new document
     with a new content digest. This includes migration, redaction, and
     adding or removing members or extensions. The new document MUST NOT
     be presented with the original's digest, or with any endorsement made
     over the original's bytes. It MUST reference the original by the
     original's content digest, and it is attributed to the party that
     transformed it (D2.7).
6. **No laundering of provenance, and no unsupported claims.**
   - A record produced by the runtime that contains host-attested or
     observed evidence attests only that the runtime recorded that
     evidence. It MUST NOT be presented as converting that evidence into
     runtime-enforced fact (I4, I5). The same will apply to any future
     signature (D4.7).
   - This ADR defines no signing, and no signing infrastructure, keys or
     trust roots exist. A document or implementation MUST NOT claim that a
     document is signed, that its authenticity is established, or that it
     conforms to DSSE or any other signing envelope. This prohibition is
     normative and holds until a later ADR defines signing. It is not part
     of the reserved provisions in D4.7.
7. **Reserved: future signatures.** This item imposes no requirement until
   a later ADR adopts it.
   - **The envelope choice is not decided by this ADR.** It remains part
     of the OQ-4 residual, and a future ADR must make it with BCP 14
     force. DSSE is the **preferred candidate**. It signs exact payload
     bytes together with a payload type, through its pre-authentication
     encoding. The payload would be the document's canonical form (D4.2),
     so that the signed bytes and the content digest refer to the same
     content.
   - Envelopes would be carried outside the documents they sign, so that
     signing never changes a content digest.
   - Which envelope is used, which parties sign which documents, key
     management, rotation, revocation and trust roots remain open (OQ-4
     residual, OQ-5).

### D5. Relationship to existing standards (OQ-23)

Proof Runtime **reuses** a small set of established specifications
normatively, **reserves** one for future use, **interoperates** with
adjacent agent and observability protocols at defined boundaries, and
**aligns** conceptually with provenance models without adopting their full
data models. The per-standard classification is in
[docs/protocols/standards-matrix.md](../protocols/standards-matrix.md).
The binding rules are:

1. **Normative reuse:**
   - RFC 8259 (JSON), RFC 7493 (I-JSON), RFC 8785 (JCS),
   - JSON Schema 2020-12,
   - SHA-256, as specified in FIPS 180-4 (the mandatory digest algorithm
     and common anchor; D4.4),
   - RFC 3339 (timestamps), RFC 4648 (base64url), RFC 3986 (URIs),
   - the in-toto `DigestSet` shape.
2. **Reserved for future use, not normative, with no choice made:** DSSE,
   as the preferred signing envelope (D4.7).
3. **Interoperability, not dependency.** MCP, A2A, CloudEvents,
   OpenTelemetry, in-toto attestations, SLSA, OIDC and SPIFFE are treated as
   external protocols that adapters may map to or from. No core protocol
   requires any of them, and no core member's meaning depends on them.
4. **No compliance claims.** Proof Runtime makes no claim of conformance,
   compatibility or certification with any external standard until a
   mapping is specified, implemented and tested against that standard's own
   conformance material.
5. **External signals are not authority.** Metadata from external protocols
   is treated as host-attested or tool-provided input, never as a CONTROL
   decision (I1, I4, I5). Examples are MCP tool annotations, A2A Agent Card
   capability declarations and telemetry attributes.

## Consequences

- The shared foundations are settled. Field-level protocol specification
  still requires two things:
  - resolution of the open questions listed in
    [docs/protocols/dependency-map.md](../protocols/dependency-map.md),
    including OQ-26 and OQ-27, which block every protocol,
  - explicit authorization of the next phase.
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
- Fail-closed behavior on unknown versions and unknown critical extensions
  means a newer producer can make an older consumer refuse a document. This
  is intended: refusal is preferred to silent divergence in either
  direction.
- Extension authors must classify their extensions correctly (D3.6). The
  [conformance cases](../protocols/conformance-cases.md) are intended to
  catch misclassification on the producer side.

## Preservation of ADR 0001

| Frozen element                                       | Effect of this ADR                                                                                                                  |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Four planes                                          | Unchanged. No plane component is added or redefined.                                                                                |
| Four core protocols                                  | Unchanged. No protocol is added; no fields are defined.                                                                             |
| I1 `MODEL != AUTHORITY`                              | Preserved: an Action IR document carries no authority; external metadata is never a CONTROL decision (D5.5).                        |
| I2 `MODEL CLAIM != VERIFIED FACT`                    | Preserved: integrity is separated from correctness (D4.1).                                                                          |
| I3 `MEMORY != POLICY`                                | Preserved: nothing in a document's encoding, version or extensions grants authority by itself (D3.3, D3.5).                        |
| I4 `TOOL OUTPUT != TRUSTED FACT`                     | Preserved: digests and records do not upgrade provenance (D4.6, D5.5).                                                              |
| I5 `HOST SUPPORT != ENFORCEMENT`                     | Preserved: external capability declarations are host-attested input (D5.5); runtime records do not launder provenance (D4.6).    |
| I6 `NO EVIDENCE -> NO VERIFIED COMPLETION`           | Preserved: references without `sha256` are invalid and mismatching digests never match (D4.4); unavailable targets are never verified (D2.7); stated digests are recomputed (D4.5).          |
| I7 `NO CAPABILITY -> NO EFFECTFUL ACTION`            | Preserved: any extension affecting capability scope is critical and fails closed when not understood (D3.4, D3.5).                 |
| I8 `PRIVILEGE EXPANSION -> EXTERNAL AUTHORIZATION`   | Preserved: no ignorable version or extension content may affect authorization (D2.5, D3.3).                                         |
| I9 `HIGH-RISK ACTION -> POLICY / APPROVAL`           | Preserved: digest-addressed documents make it possible to bind an approval to exact content; whether approvals must do so is OQ-9. |
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
  concerns (design constraint C6). How are these decision records
  identified and carried, and how do later records reference them?
  - The alternatives and their security implications are analyzed in
    [foundations-analysis.md § OQ-29](../protocols/foundations-analysis.md#oq-29-open-control-decision-records).
  - Whatever the outcome, design constraint C17 applies: a record that
    describes a decision is never itself an authorization credential.
  - This ADR does not select a design, and the question must not be
    resolved by adding a fifth core protocol without a new ADR that amends
    ADR 0001.

The residual part of **OQ-4** stays open: which signature envelope is used
(DSSE is the preferred candidate), which parties sign which documents, and
how keys are managed, rotated and revoked. It depends on OQ-5.

## Verification status of cited standards

The Claude session that drafted this ADR ran with restricted network
access. Some primary sources were read directly in that session.

For RFC 8259, 7493, 8785, 7515, 3339, 4648 and 3986, JSON Schema 2020-12,
W3C PROV-DM, SLSA v1.2, OpenID Connect Core and FIPS 180-4, the repository
owner reported on 2026-10-02 that an independent review accessed the
official sources. The drafting session itself could not access them.

The [standards matrix](../protocols/standards-matrix.md) records the
official link and verification basis of each citation, and the items that
remain. No verification implies implementation conformance with any
standard (D5.4).
