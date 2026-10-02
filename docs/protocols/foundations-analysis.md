# Shared Protocol Foundations: Analysis and Recommendations

> **Status: Proposed**, supporting
> [ADR 0002](../adr/0002-shared-protocol-foundations.md). This document
> explains the reasoning behind the decisions. The decisions themselves
> (D1 to D5) are stated only in the ADR. If the two disagree, the ADR
> governs, and the disagreement is a defect.

Each section covers one open question from
[ADR 0001](../adr/0001-architecture-freeze.md#open-questions):

1. the engineering requirements,
2. the alternatives considered,
3. interoperability,
4. security consequences,
5. migration and extension implications,
6. the recommendation,
7. residual risks and future work.

Standards citations and their verification status are in the
[standards matrix](standards-matrix.md). Several primary sources could not
be reached during this review. Statements below that depend on them are
marked **(unverified)**.

## OQ-1: Encoding and schema language

### Requirements

- **R1.1 Language neutrality.** Any mainstream language can parse and
  produce documents without generated code or a proprietary toolchain.
- **R1.2 Unambiguous parsing.** Two conformant parsers must reach the same
  data model from the same bytes. This rules out "last one wins" duplicate
  keys and silent number truncation.
- **R1.3 Deterministic digests.** A canonical form must exist, so that
  documents can be referenced by digest across implementations (OQ-4).
- **R1.4 Machine validation.** Structure can be validated before semantic
  processing, with schemas that are themselves portable.
- **R1.5 Reviewability.** Documents are audit records. People reviewing
  incidents need to read them without special tools.
- **R1.6 Bounded resource use.** Parsing untrusted documents must not allow
  resource exhaustion.

### Alternatives

| Option | Strengths | Weaknesses for Proof Runtime |
|---|---|---|
| **JSON + I-JSON profile + JSON Schema 2020-12** | Universal parsers. Readable. Mature schema ecosystem. A canonicalization scheme exists (JCS). Used by in-toto, A2A, CloudEvents' JSON format and MCP. | Default parsers often accept duplicate keys and lose precision on large numbers, so a strict profile is required. More verbose than binary formats. |
| **CBOR + CDDL** (RFC 8949, RFC 8610) | Compact. Deterministic encoding defined in the CBOR RFC **(unverified)**. Native binary values. | Not human-readable. Smaller tooling base. CDDL validators are less widespread than JSON Schema validators. |
| **Protocol Buffers** | Compact and fast, with strong code generation. | Requires generated code. Deterministic serialization is not guaranteed across languages. Unknown-field handling varies. Poor fit for readable audit records. |
| **JSON-LD / RDF** | Rich semantics. Aligns with W3C PROV. | Heavy processing model. RDF canonicalization adds complexity and attack surface. Disproportionate for v0.1. |
| **YAML** | Readable. | Multiple parsing ambiguities and implicit typing. Unsuitable for security-relevant interchange. |

### Interoperability

JSON matches the wire formats of the adjacent protocols evaluated:
in-toto Statements, A2A, CloudEvents' JSON format and MCP **(MCP's
JSON-RPC format unverified)**. Mapping to them needs no transcoding.

### Security

- **Duplicate keys.** Duplicate-key acceptance can make two components see
  different documents (a parser-differential attack). D1 therefore requires
  rejection rather than relying on default parser behavior.
- **Large numbers.** Integers above 2^53 lose precision in IEEE 754 doubles
  in many languages. Floating-point values serialize differently across
  languages. D1 bans non-integer core numbers and bounds integer range.
- **Remote schema references.** Resolving `$ref` over the network during
  validation would let documents trigger outbound requests. D1 requires
  offline, bundled schemas.
- **Resource exhaustion.** Deeply nested or very large documents can
  exhaust resources. D1 requires every protocol to define limits (OQ-27).

### Migration and extension

JSON objects extend naturally by adding members. Combined with D2 and D3,
this supports minor-version additions and namespaced extensions. If a
binary encoding is needed later, CBOR can carry the JSON data model, but
only through a new ADR that defines the mapping (D1.5).

### Recommendation

Adopt **D1**: JSON under a strict I-JSON-based profile, validated by
offline JSON Schema 2020-12 schemas that are a normative minimum, with
prose specifications normative.

### Residual risks and future work

- Strict profile enforcement depends on implementations choosing strict
  parsers. Conformance test vectors must include duplicate keys, a
  byte order mark, out-of-range integers, non-integer numbers and lone
  surrogates.
- OQ-27 must set concrete limits before any protocol is specified.

## OQ-2: Versioning and compatibility

### Requirements

- **R2.1** Each protocol can evolve without forcing the others to change.
- **R2.2** A consumer can tell, before interpreting content, whether it is
  able to process a document.
- **R2.3** Version skew between producers and consumers must never weaken
  authorization or verification (I7, I8, I6).
- **R2.4** Historical documents remain verifiable after the protocol
  evolves. They are evidence.

### Alternatives

| Option | Assessment |
|---|---|
| **Single global version** for all four protocols | Couples unrelated protocol changes. Rejected (R2.1). |
| **Full SemVer (`MAJOR.MINOR.PATCH`) in documents** | Patch releases are editorial and do not change the wire meaning, so carrying them invites spurious incompatibility checks. |
| **Date-based versions** (as MCP uses) | Clear for negotiated sessions, but they do not express compatibility. Proof Runtime documents are stored and read years later, without negotiation. |
| **Type URI with major version, plus `MAJOR.MINOR`** (in-toto-like type URI) | The type identifier fixes the meaning class. The minor version supports additive evolution. Chosen. |

### Interoperability

- in-toto encodes the major version in its type URIs and uses SemVer for
  its types; D2 follows the same pattern.
- CloudEvents carries a `specversion` attribute; D2's protocol version plays
  the equivalent role.
- No negotiation protocol is needed. Documents are self-describing, and
  transport-level negotiation (such as MCP's) stays the transport's
  concern.

### Security

The main risk is a version-skew downgrade. An older consumer ignores a new
member that was meant to restrict something, and so it allows what a newer
consumer would deny.

D2.5 adopts the monotonic principle from the in-toto attestation framework:
an addition that matters for security cannot be ignorable. Such additions
must be introduced as a new major version (which older consumers reject) or
as a critical extension (which older consumers refuse).

### Migration and extension

- Documents are immutable and digest-addressed (D2.7). Migration therefore
  creates new documents that reference the originals.
- Verification of historical records stays possible as long as
  implementations retain support for reading old major versions. How long
  that support must last is a future policy question.
- Pre-1.0 versions carry no compatibility promise, which keeps early
  iteration cheap.

### Recommendation

Adopt **D2**: independent per-protocol versions, a type URI that carries
the major version, and `MAJOR.MINOR` in documents. Unknown majors fail
closed, and security-relevant additions are never ignorable.

### Residual risks and future work

- **OQ-26.** The type URI namespace needs a durable authority under project
  control.
- A support window for reading historical major versions should be decided
  before 1.0.

## OQ-3: Extension model

### Requirements

- **R3.1** Domain-specific and host-specific data can be added without
  changing core protocols (ADR 0001 §2; industry neutrality).
- **R3.2** Independent parties can define extensions without collisions,
  and without a central registry in v0.1.
- **R3.3** A consumer can tell extensions it may safely ignore from
  extensions it must understand.
- **R3.4** Unknown extensions never silently weaken authorization or
  verification.

### Alternatives

| Option | Assessment |
|---|---|
| **Unprefixed additional members anywhere** | Collides with future core members. Cannot express criticality. Rejected. |
| **Reverse-DNS names** (`com.example.foo`) | Collision-resistant, but not dereferenceable and not a common convention in the adjacent protocols. |
| **URI-identified extensions in a dedicated container, with a critical list** | Matches established patterns: JWS `crit` **(unverified)**, X.509's critical flag **(unverified)**, A2A extension URIs with `required` (verified). Chosen. |
| **Central registry** | Strong governance, but premature for a pre-alpha project. Deferred. |

### Interoperability

- A2A (verified) identifies extensions by URI and lets an agent declare an
  extension `required`; clients that lack it receive an
  `ExtensionSupportRequiredError`. D3's critical list plays the same role
  at the document level, rather than in session negotiation.
- CloudEvents (verified) recommends ignoring unknown extension attributes
  and defines no criticality. CloudEvents is therefore suitable only as a
  transport, not as the extension mechanism.

### Security

- **Relaxing extensions.** The main hazard is an extension that relaxes a
  control, such as "also allow X". A consumer that does not recognize it
  would deny, which is safe. But the wording of the rule matters: D3.5
  requires every relaxing extension to be marked critical. A consumer that
  doesn't understand it then refuses the whole document, rather than
  honoring part of it or guessing.
- **Stripping.** An intermediary could remove an extension or its critical
  marker. Digests detect this only if the consumer holds an independently
  obtained digest. Authenticity (signatures) is needed to detect it in
  general. This residual risk is stated in D3.6.
- **Placement.** Keeping extensions in one designated element prevents
  extension data from being confused with, or shadowing, core members.

### Migration and extension

- Extensions version themselves through the major version in their URI.
- An extension that proves broadly useful can be promoted into a core
  minor version. It may be promoted as an ignorable member only if
  ignoring it is safe (D2.5).

### Recommendation

Adopt **D3**: a single extensions element keyed by URI, an explicit
per-document critical list, fail closed on unknown critical extensions,
the monotonic rule for non-critical extensions, and industry semantics only
in extensions.

### Residual risks and future work

- Criticality depends on extension authors marking it honestly. Conformance
  tests and review guidance for extensions are needed.
- Stripping protection depends on the signing design (OQ-4 residual, OQ-5).
- An extension registry or index may be warranted after 1.0.

## OQ-4: Canonicalization and integrity

### Requirements

- **R4.1** Documents can reference other documents and artifacts by digest,
  with the same digest regardless of which implementation serialized them.
- **R4.2** Integrity, authenticity, authorization and correctness are never
  conflated (I2, I5, I6).
- **R4.3** The design leaves room for signatures without requiring signing
  infrastructure in v0.1.
- **R4.4** Algorithm agility: digest algorithms can be replaced without a
  protocol major version.

### Alternatives

| Option | Assessment |
|---|---|
| **Digest the exact bytes as received** | Simplest, and no canonicalization bugs. But semantically identical documents from different serializers produce different digests, which breaks cross-implementation references (R4.1). Still the right choice for opaque artifacts. |
| **JCS canonical form, then digest** | Deterministic across implementations for I-JSON input (verified, author's repository). Multiple language implementations exist. A2A uses JCS before signing Agent Cards (verified). Chosen for documents. |
| **Deterministic CBOR** | Only applicable with CBOR encoding. Deferred together with D1.5. |
| **JWS (compact or JSON serialization) as the integrity carrier** | Couples integrity to signing. A JWS without a signature would be misleading. Kept as a possible later alternative for signatures. |
| **DSSE envelope for signatures** | Signs exact bytes plus payload type through PAE (verified), avoiding re-canonicalization during verification. Supports multiple signatures. Used by in-toto. Chosen as the future envelope. |

### Interoperability

- The in-toto `DigestSet` shape (verified) makes Proof Runtime digest
  references directly usable in in-toto subjects.
- DSSE is the envelope in-toto uses, which eases a future export of Evidence
  Receipts as attestations.
- JCS alignment with A2A means a JWS-over-JCS signature profile could be
  added later for parity with A2A Agent Cards.

### Security

- **JCS limits.** JCS does not normalize Unicode, and it serializes numbers
  the way ECMAScript does (verified, author's repository). D1's integer-only
  core numbers and ASCII identifiers remove the main sources of
  cross-language divergence. Strings in content are digested as given;
  differently normalized strings are, correctly, different content.
- **No algorithm, no match.** A digest reference with no acceptable
  algorithm is treated as unverifiable. It is never a match. This prevents
  downgrade to weak or unknown algorithms.
- **What crypto does not prove.** Hashing proves only that bytes are
  unchanged. Signing proves only that a key holder endorsed bytes. Neither
  proves that execution happened as described, that the signer was
  authorized, or that the content is true (D4.1, C11).
- **Laundering.** A runtime signature over a receipt that contains
  host-attested evidence would be dangerous if it were read as upgrading
  that evidence. D4.6 prohibits presenting it that way.

### Migration and extension

- Algorithm agility comes from `DigestSet` (D4.4).
- Signing can be added later without changing document content digests,
  because envelopes are external (D4.5).

### Recommendation

Adopt **D4**:

- the JCS canonical form for document digests,
- exact-bytes digests for opaque artifacts,
- `DigestSet`-shaped digests with `sha256` mandatory to support,
- DSSE as the designated future signing envelope,
- an explicit separation of integrity, authenticity, authorization and
  correctness.

### Residual risks and future work

- **Signing design.** Who signs what, key management, rotation, revocation
  and trust roots remain open (OQ-4 residual). They depend on the identity
  model (OQ-5).
- **Test vectors.** Shared JCS and digest test vectors must be produced and
  run against at least two independent implementations before JCS-based
  digests are relied on.
- **Trusted time.** Whether some records need trusted time is OQ-28.
- **Audit integrity.** Hash chaining and transparency logs are OQ-20.

## OQ-23: Relationship to existing standards

### Requirements

- **R23.1** Reuse proven building blocks instead of inventing new ones,
  wherever they meet the requirements above.
- **R23.2** Interoperate with the agent, tool, observability and
  attestation ecosystems at clear boundaries.
- **R23.3** Never import another protocol's trust assumptions into the
  CONTROL or TRUST planes (I1, I4, I5).
- **R23.4** Make no compliance claims without implementation and
  conformance testing.

### Assessment

The full per-standard evaluation is in the
[standards matrix](standards-matrix.md). In summary:

- **Reuse:** JSON, I-JSON, JSON Schema 2020-12, JCS, SHA-256, in-toto
  `DigestSet`, DSSE (future), RFC 3339, RFC 4648 and RFC 3986. These are
  small, stable building blocks that directly meet the OQ-1 and OQ-4
  requirements.
- **Interoperate:** MCP, A2A, CloudEvents, OpenTelemetry, in-toto
  attestations, JWS, OIDC and SPIFFE. These protocols describe adjacent
  concerns: tool invocation, agent messaging, event transport, telemetry,
  attestations and identity. Proof Runtime maps to them at adapter
  boundaries but does not depend on them, because their trust models differ
  from the invariants. For example, tool annotations and Agent Card
  capability declarations are self-reported. And telemetry has no
  integrity.
- **Align:** W3C PROV's entity / activity / agent model, for Evidence
  Receipt provenance vocabulary.
- **Out of scope for v0.1:** CBOR, CDDL, COSE, Protocol Buffers, W3C
  Verifiable Credentials, and SLSA in the core.

### Security

Each adjacent protocol has its own trust model. The key rule (D5.4) is that
signals from them enter Proof Runtime only as host-attested or
tool-provided input, never as CONTROL decisions or verified facts.

### Recommendation

Adopt **D5**.

### Residual risks and future work

- The items listed under "Items to verify before acceptance" in the
  standards matrix must be checked against primary sources.
- Each interoperability mapping (MCP, A2A, in-toto export) needs its own
  specification and tests before any compatibility claim is made.
- The OpenTelemetry GenAI conventions are at Development stability
  (secondary source). Any mapping to them is expected to change.
