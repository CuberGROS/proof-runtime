# Standards Compatibility and Reuse Matrix

> **Status: Proposed**, as part of
> [ADR 0002](../adr/0002-shared-protocol-foundations.md). It answers OQ-23 for
> v0.1. Nothing here claims conformance or compatibility with any standard;
> see [ADR 0002 § D5](../adr/0002-shared-protocol-foundations.md#d5-relationship-to-existing-standards-oq-23).

## Dispositions

| Disposition      | Meaning                                                                                                                                  |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Reuse**        | Used normatively by the core protocols in v0.1.                                                                                         |
| **Interoperate** | External protocol. Adapters may map Proof Runtime documents to or from it at a defined boundary. The core protocols do not depend on it. |
| **Align**        | Its concepts inform Proof Runtime's vocabulary or design. Its data model is not adopted in v0.1.                                         |
| **Out of scope** | Not used in v0.1. It may be reconsidered by a later ADR.                                                                                |

## Verification status

The research behind this matrix ran with restricted network access. Each
row records how its key claims were checked:

- **Verified (primary):** read in this review from the standard's own
  specification or official repository.
- **Verified (secondary):** checked in this review only against secondary
  sources.
- **Not verified in this review:** the primary source was unreachable from
  the review environment (the RFC Editor, IETF, W3C, `modelcontextprotocol.io`
  and `a2a-protocol.org` sites were blocked). The claim comes from prior
  knowledge and must be checked before ADR 0002 is accepted.

## Matrix

### Encoding, schema and integrity building blocks

| Standard | Disposition | What Proof Runtime takes | What it does not take, and caveats | Verification |
|---|---|---|---|---|
| **JSON** (RFC 8259) | Reuse | Interchange encoding for all four protocols (D1.1). | — | Not verified in this review. |
| **I-JSON** (RFC 7493) | Reuse | Interoperability profile: UTF-8, unique member names, number-range guidance (D1.2). | Proof Runtime adds stricter rules: integer-only core numbers, no `null` for absence, ASCII identifiers. | Not verified in this review. |
| **JSON Schema 2020-12** | Reuse | Structural validation schemas for each protocol (D1.3). | `format` is not relied on for security checks; no network `$ref` resolution. Schema validity is necessary, not sufficient. | Verified (primary): json-schema.org states 2020-12 is the current version. |
| **JCS** (RFC 8785) | Reuse | Canonical form for content digests (D4.2). | No Unicode normalization (inputs must already be stable). Numbers use ECMAScript serialization, which is one reason D1 bans non-integer core numbers. The RFC's IETF category was not confirmed here. | Verified (primary, author's repository): I-JSON input, sorted keys, ECMAScript number serialization, no Unicode normalization, implementations in Rust, JavaScript, Java, Go, .NET, Python and others. RFC text not reachable. |
| **SHA-256** (FIPS 180-4) | Reuse | Mandatory-to-support digest algorithm (D4.4). | Algorithm agility through `DigestSet`. | Not verified in this review (standard algorithm, widely implemented). |
| **in-toto `DigestSet`** (Attestation Framework v1) | Reuse (shape only) | Digest representation: algorithm name → lowercase hex (D4.4). Consumers accept only algorithms they consider secure and ignore unrecognized ones. | Proof Runtime adds: a reference with no acceptable algorithm is unverifiable, never a match. | Verified (primary): in-toto attestation `digest_set.md`. |
| **DSSE** (Dead Simple Signing Envelope) | Reuse (future) | Designated envelope for future signatures (D4.5): signs exact bytes plus payload type through its pre-authentication encoding (PAE). | Not implemented in v0.1. Key management and trust roots are open (OQ-4 residual, OQ-5). | Verified (primary): DSSE `protocol.md`, PAE definition. |
| **RFC 3339** timestamps, **RFC 4648** base64url, **RFC 3986** URIs | Reuse | Timestamp format, embedded-bytes encoding, identifier syntax (D1.2, D3.2). | Timestamps are producer claims, not trusted time (OQ-28). | Not verified in this review. |
| **JWS** (RFC 7515) | Interoperate | Its `crit` header is the model for D3.4's critical-extension list. JWS with JCS may be offered later as an alternative to DSSE, for example to interoperate with A2A Agent Card signatures. | Not the designated envelope in v0.1. | Not verified in this review (RFC unreachable). A2A's use of JWS with JCS was verified (see A2A row). |
| **CBOR** (RFC 8949), **CDDL** (RFC 8610), **COSE** (RFC 9052) | Out of scope | — | A compact binary encoding may be added later by ADR, with a lossless mapping from the JSON data model (D1.5). | Not verified in this review. |
| **Protocol Buffers** | Out of scope | — | Unknown-field and deterministic-serialization behavior varies by language. Requires generated code. | Not verified in this review. |

### Agent and tool protocols

| Standard | Disposition | What Proof Runtime takes | What it does not take, and caveats | Verification |
|---|---|---|---|---|
| **MCP** (Model Context Protocol) | Interoperate | A host adapter may turn an MCP tool call into an Action IR proposal, and record MCP tool results as tool-provided evidence (I4). | MCP metadata, such as tool annotations, is input, never a CONTROL decision (D5.4). MCP's date-based protocol versions are not tied to Proof Runtime versions. | Verified (primary, official repository): schema versions are date-based directories, with `2026-07-28` the newest non-draft listed. The wire format (JSON-RPC 2.0) and the annotation trust guidance were not verified here; the documentation site was unreachable. |
| **A2A** (Agent2Agent) | Interoperate | A2A carries messages between agents; Proof Runtime documents can travel as A2A data. Precedents that support ADR 0002: extensions identified by URI with a `required` flag (cf. D3), and Agent Cards canonicalized with JCS before JWS signing (cf. D4.2). | An A2A `Task` is a lifecycle object managed by a remote agent. It is not a Task Capsule. A2A capability declarations are host-attested (I5). | Verified (primary, official repository): Linux Foundation project; specification version 1.0.0; JSON-RPC 2.0, gRPC and HTTP+JSON bindings; `required` extensions trigger `ExtensionSupportRequiredError`; Agent Card signing uses JWS after JCS. |
| **CloudEvents** 1.0 | Interoperate | Optional transport envelope for emitting protocol documents as events. | CloudEvents does not define integrity or signing, and recommends ignoring unknown extension attributes. Neither property can substitute for D3 or D4. | Verified (primary, official repository): `specversion` `1.0`; required `id`, `source`, `specversion`, `type`; lowercase alphanumeric extension names; no signing. |

### Observability, provenance and supply chain

| Standard | Disposition | What Proof Runtime takes | What it does not take, and caveats | Verification |
|---|---|---|---|---|
| **OpenTelemetry**, including the GenAI semantic conventions | Interoperate | Optional correlation of protocol documents with traces, for example by carrying trace identifiers. | Telemetry is not evidence by default. It has no integrity protection, and the GenAI conventions are not stable. | Verified (secondary only): as of mid-2026, all `gen_ai.*` conventions are at "Development" stability and moved to a separate repository in v1.42.0. Official repositories were reachable but did not state status on the pages read. |
| **W3C PROV** (PROV-DM, PROV-O) | Align | Its entity / activity / agent distinction informs how Evidence Receipts express provenance (OQ-17, OQ-21). | No RDF or PROV-O serialization in v0.1. | Not verified in this review (W3C site unreachable). |
| **in-toto Attestation Framework** v1 (Statement, predicate) | Interoperate | A future exporter may present Evidence Receipts as in-toto Statements with a Proof Runtime predicate type. Its "monotonic principle" (ignoring fields must never turn denial into allowance) is adopted in D2.5 and D3.5. | Subjects are immutable artifacts identified by digest. Proof Runtime receipts also describe actions and decisions, so the mapping must be specified, not assumed. | Verified (primary): Statement `_type` `https://in-toto.io/Statement/v1`; subjects require a digest; consumers ignore unrecognized fields; monotonic principle. |
| **SLSA** | Out of scope (core); Interoperate (software-engineering extensions) | SLSA provenance documents may appear as evidence artifacts, referenced by digest, in software-engineering extensions. | SLSA is scoped to software supply-chain integrity. It must not shape the industry-neutral core (D3.7). | Not verified in this review: current release number and provenance predicate URI not confirmed. |

### Identity

| Standard | Disposition | What Proof Runtime takes | What it does not take, and caveats | Verification |
|---|---|---|---|---|
| **OpenID Connect** | Interoperate (deferred to OQ-5) | Candidate source of human and service principal identity. | An identity assertion establishes who, not what that principal is authorized to do. It is not a capability. | Not verified in this review. |
| **SPIFFE** (SPIFFE ID, SVID) | Interoperate (deferred to OQ-5) | Candidate workload identity for runtime and host components (`spiffe://trust-domain/path`). | Workload identity is not authorization. Choosing SPIFFE is OQ-5's decision. | Verified (primary, official repository): ID format, trust domain, X.509-SVID, JWT-SVID and WIT-SVID. |
| **W3C Verifiable Credentials** | Out of scope | Possible future mechanism for delegable capabilities (OQ-7). | — | Not verified in this review. |

## Items to verify before acceptance

Before ADR 0002 is accepted, check these against their primary sources:

1. RFC 8785: confirm its IETF category and its exact rules for property
   ordering and number serialization.
2. RFC 7493: confirm the exact I-JSON requirements for duplicate names,
   number ranges and surrogates.
3. RFC 7515 §4.1.11: confirm `crit` semantics, used as the model for D3.4.
4. MCP: confirm the wire format and the specification's guidance on whether
   tool annotations are trusted.
5. SLSA: confirm the current release and provenance predicate type.
6. W3C PROV: confirm its recommendation status.
7. JSON Schema 2020-12: confirm that `format` is annotation-only by default
   (the basis of D1.3's rule against relying on it). Confirm also the
   behavior of `$id`-based offline resolution in the chosen validators.
