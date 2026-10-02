# Standards Compatibility and Reuse Matrix

> **Status: Accepted**, as part of
> [ADR 0002](../adr/0002-shared-protocol-foundations.md). It answers OQ-23 for
> v0.1. Nothing here claims conformance or compatibility with any standard;
> see [ADR 0002 § D5](../adr/0002-shared-protocol-foundations.md#d5-relationship-to-existing-standards-oq-23).

## Dispositions

| Disposition      | Meaning                                                                                                                                  |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Reuse**        | Used normatively by the core protocols in v0.1.                                                                                         |
| **Reserved**     | Designated as the preferred direction for future work. Not normative in v0.1, and no conformance is implied.                            |
| **Interoperate** | External protocol. Adapters may map Proof Runtime documents to or from it at a defined boundary. The core protocols do not depend on it. |
| **Align**        | Its concepts inform Proof Runtime's vocabulary or design. Its data model is not adopted in v0.1.                                         |
| **Out of scope** | Not used in v0.1. It may be reconsidered by a later ADR.                                                                                |

## Verification levels

The Claude session that prepared this matrix ran with restricted network
access. The RFC Editor, IETF Datatracker, W3C, `openid.net`, `slsa.dev`,
`csrc.nist.gov`, `modelcontextprotocol.io` and `a2a-protocol.org` were
blocked by its egress proxy. Each claim below carries one or more of these
levels:

- **Verified (primary).** Read in this review from the official
  specification or the standard's official source repository.
- **Corroborated.** The official page was located by web search, and the
  search engine's excerpt of it supports the claim. The full text was not
  retrieved. These claims must be re-checked against the full text before
  ADR 0002 is accepted.
- **Independent review (owner-reported).** The repository owner reported
  on 2026-10-02 that an independent review accessed the listed official
  source and supports the row's claims. This session did not access that
  source, and does not claim to have verified it.
- **Secondary.** Supported only by third-party sources.
- **Unverified.** Not checked in this review; stated from prior knowledge.

## Matrix

### Encoding, schema and integrity building blocks

| Standard (official source) | Disposition | What Proof Runtime takes | Caveats | Verification |
|---|---|---|---|---|
| JSON — [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259) | Reuse | Interchange encoding (D1.1). | — | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. |
| I-JSON — [RFC 7493](https://www.rfc-editor.org/rfc/rfc7493) | Reuse | Profile base (D1.2). | Proof Runtime is stricter: integer-only numbers throughout the document (no fraction, exponent or `-0`), no `null` for absence, ASCII identifiers. | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. **Also corroborated in this session (search excerpt):** "Objects in I-JSON messages MUST NOT have members with duplicate names"; integers outside [−(2^53)+1, (2^53)−1] cannot be expected to be treated as exact. **Unverified:** the RFC's exact wording on surrogates and its protocol-design recommendations, beyond the independent review. |
| JSON Schema 2020-12 — [Validation §7.2](https://json-schema.org/draft/2020-12/json-schema-validation); [version status](https://json-schema.org/specification) | Reuse | Structural validation (D1.3). | `format` is not relied on for security checks. Schemas resolve offline. Validity is necessary, not sufficient. | **Verified (primary):** 2020-12 is the current version. Under the Format-Annotation vocabulary, `format` "MUST be collected as an annotation", and optional assertion "MUST be disabled by default" (§7.2.1). **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. **Unverified:** offline `$id` resolution behavior in specific validators. |
| JCS — [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785) (§3.1 input data, including numbers; §3.2.3 property sorting); [author's repository](https://github.com/cyberphone/json-canonicalization) | Reuse | Canonical form for content digests (D4.2). D1.2's numeric profile is deliberately stricter than its §3.1 input requirements: integers only. | Informational, Independent Submission: the RFC Editor makes no statement about its value for implementation. No Unicode normalization. ECMAScript-style number serialization. | **Corroborated:** Informational, Independent Submission, June 2020; builds on ECMAScript serialization, the I-JSON subset and deterministic property sorting. **Verified (primary, author's repository):** I-JSON input; no Unicode normalization; implementations in Rust, JavaScript, Java, Go, .NET, Python and others. **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. This covers §3.1 (input data, including the requirement that numbers be expressible as IEEE 754 doubles) and §3.2.3 (property names sorted by UTF-16 code units). This session's own evidence for those two sections was limited to third-party implementation documentation (secondary). |
| SHA-256 — FIPS 180-4, Secure Hash Standard ([NIST CSRC](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)) | Reuse | Mandatory-to-support digest, and the mandatory entry in every v0.1 reference (D4.4). | Algorithm agility through `DigestSet`; `sha256` is the common anchor. | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. The exact deep link was not checked by this session. |
| in-toto `DigestSet` — [digest_set.md](https://github.com/in-toto/attestation/blob/main/spec/v1/digest_set.md) | Reuse (shape only) | Digest representation (D4.4). | Proof Runtime is stricter in three ways. Every v0.1 reference must contain `sha256`, which every consumer verifies as a common anchor. **All** other accepted entries must also match the same content, where in-toto treats a set as matching if any acceptable entry matches. And a reference without `sha256` is invalid and rejected. | **Verified (primary):** algorithm name → lowercase hex; consumers "MUST only accept algorithms that they consider secure and MUST ignore unrecognized or unaccepted algorithms"; `sha256` recommended; "Two DigestSets SHOULD be considered matching if ANY acceptable field matches". |
| DSSE — [protocol.md](https://github.com/secure-systems-lab/dsse/blob/master/protocol.md) | **Reserved** | Preferred candidate for a future signing envelope (D4.7). The envelope choice itself remains open (OQ-4 residual). | Not normative in v0.1. No signing infrastructure exists. No DSSE conformance is claimed. | **Verified (primary):** signs `PAE(type, body)` = `"DSSEv1" SP LEN(type) SP type SP LEN(body) SP body`; `payloadType` identifies interpretation. |
| RFC 3339, RFC 4648, RFC 3986 — [3339](https://www.rfc-editor.org/rfc/rfc3339), [4648](https://www.rfc-editor.org/rfc/rfc4648), [3986](https://www.rfc-editor.org/rfc/rfc3986) | Reuse | Timestamps, base64url, URI syntax (D1.2, D3.2). | Timestamps are producer claims (OQ-28). | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. |
| JWS — [RFC 7515 §4.1.11](https://www.rfc-editor.org/rfc/rfc7515#section-4.1.11) | Interoperate | Its `crit` header is the model for D3.4. JWS over JCS is a possible later alternative envelope, for example for parity with A2A Agent Cards. | Not the designated envelope. | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. **Also corroborated in this session (search excerpt):** `crit` lists extensions that "MUST be understood and processed"; if any is not understood and supported by the recipient, the JWS is invalid. Several 2026 library vulnerabilities (for example CVE-2026-32597 in PyJWT) concern implementations that failed to enforce this (secondary). |
| CBOR, CDDL, COSE — [RFC 8949](https://www.rfc-editor.org/rfc/rfc8949), [RFC 8610](https://www.rfc-editor.org/rfc/rfc8610), [RFC 9052](https://www.rfc-editor.org/rfc/rfc9052) | Out of scope | — | May be added later by ADR (D1.5). | Unverified. |
| Protocol Buffers | Out of scope | — | Deterministic serialization and unknown-field handling vary by language. | Unverified. |

### Agent and tool protocols

| Standard (official source) | Disposition | What Proof Runtime takes | Caveats | Verification |
|---|---|---|---|---|
| MCP — [schema source](https://github.com/modelcontextprotocol/modelcontextprotocol/tree/main/schema) | Interoperate | Adapters may map MCP tool calls to Action IR proposals, and record tool results as tool-provided evidence (I4). | Tool annotations are input, never a CONTROL decision (D5.5). | **Verified (primary, [2026-07-28 schema.ts](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2026-07-28/schema.ts)):** `LATEST_PROTOCOL_VERSION` is `"2026-07-28"`. ToolAnnotations: "all properties in `ToolAnnotations` are **hints**. They are not guaranteed to provide a faithful description of tool behavior … Clients should never make tool use decisions based on `ToolAnnotations` received from untrusted servers." **Verified (primary, [2025-11-25 schema.ts](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/schema/2025-11-25/schema.ts)):** `JSONRPC_VERSION` is `"2.0"`. |
| A2A — [specification.md](https://github.com/a2aproject/A2A/blob/main/docs/specification.md) | Interoperate | Transport for Proof Runtime documents between agents. Its URI-identified extensions with `required`, and its JCS-before-JWS Agent Card signing, are precedents for D3 and D4. | An A2A `Task` is not a Task Capsule. Agent Card declarations are host-attested (I5). | **Verified (primary):** Linux Foundation project; version 1.0.0; JSON-RPC 2.0, gRPC and HTTP+JSON bindings; `required` extensions produce `ExtensionSupportRequiredError`; Agent Cards "MUST be canonicalized using … JCS" before JWS signing. |
| CloudEvents 1.0 — [spec.md](https://github.com/cloudevents/spec/blob/main/cloudevents/spec.md) | Interoperate | Optional event transport. | No integrity or signing. Unknown extension attributes are recommended to be ignored, so CloudEvents cannot carry D3 criticality. | **Verified (primary):** `specversion` `1.0`; required `id`, `source`, `specversion`, `type`; lowercase alphanumeric extension names; no signing defined. |

### Observability, provenance and supply chain

| Standard (official source) | Disposition | What Proof Runtime takes | Caveats | Verification |
|---|---|---|---|---|
| OpenTelemetry GenAI semantic conventions — [repository](https://github.com/open-telemetry/semantic-conventions-genai) | Interoperate | Optional trace correlation. | Telemetry is not evidence by default. The conventions are not stable. | **Verified (primary):** a dedicated repository exists, covering GenAI clients, MCP and provider conventions. **Secondary:** all `gen_ai.*` conventions are at "Development" stability as of mid-2026. |
| W3C PROV-DM — [W3C TR](https://www.w3.org/TR/prov-dm/) | Align | The entity / activity / agent distinction, for Evidence Receipt provenance vocabulary (OQ-17, OQ-21). | No RDF or PROV-O serialization in v0.1. | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. **Also corroborated in this session (search excerpt):** W3C Recommendation dated 30 April 2013. Entity: "a physical, digital, conceptual, or other kind of thing with some fixed aspects". Activity: "something that occurs over a period of time and acts upon or with entities". Agent: "something that bears some form of responsibility for an activity taking place, for the existence of an entity, or for another agent's activity". |
| in-toto Attestation Framework v1 — [README](https://github.com/in-toto/attestation/blob/main/spec/v1/README.md), [Statement](https://github.com/in-toto/attestation/blob/main/spec/v1/statement.md) | Interoperate | A future exporter may present Evidence Receipts as in-toto Statements. D2.5 and D3.5 extend its monotonic principle to both directions. | Subjects are digest-identified artifacts; receipts also describe actions and decisions, so a mapping must be specified. | **Verified (primary):** `_type` `https://in-toto.io/Statement/v1`; each subject "MUST have `digest` set"; "Consumers MUST ignore unrecognized fields unless otherwise noted"; monotonic principle. |
| SLSA — [v1.2 specification](https://slsa.dev/spec/v1.2/); [v1.2 provenance](https://slsa.dev/spec/v1.2/provenance); [v1.2 announcement](https://slsa.dev/blog/2025/11/announce-slsa-v1.2) | Out of scope (core); Interoperate (software-engineering extensions) | SLSA provenance may appear as digest-referenced evidence in software-engineering extensions. | Software supply-chain scope. It must not shape the industry-neutral core (D3.8). | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. **Also corroborated in this session (search excerpt):** v1.2 is the current published version (announced November 2025); it adds a Source Track and is described as backwards compatible with v1.1. The official GitHub repository has no tagged releases. The provenance `predicateType` URI is defined on the official v1.2 provenance page; it is not restated here because this session could not read that page. |

### Identity

| Standard (official source) | Disposition | What Proof Runtime takes | Caveats | Verification |
|---|---|---|---|---|
| OpenID Connect Core 1.0 — [specification](https://openid.net/specs/openid-connect-core-1_0.html) | Interoperate (deferred to OQ-5) | Candidate source of human and service principal identity. | OIDC authenticates an End-User; it does not authorize actions. An ID Token is not a capability (C7). | **Independent review (owner-reported):** the official source was accessed by an independent reviewer, as reported by the repository owner on 2026-10-02, in support of this row's claims. This Claude session could not access it. **Also corroborated in this session (search excerpt):** "OpenID Connect Core 1.0 incorporating errata set 2", 15 December 2023; "a simple identity layer on top of the OAuth 2.0 protocol" that lets clients "verify the identity of the End-User based on the authentication performed by an Authorization Server". |
| SPIFFE — [SPIFFE-ID.md](https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE-ID.md) | Interoperate (deferred to OQ-5) | Candidate workload identity for runtime and host components. | Workload identity is not authorization. | **Verified (primary):** `spiffe://trust-domain/path`; X.509-SVID, JWT-SVID and WIT-SVID formats. |
| W3C Verifiable Credentials | Out of scope | Possible future mechanism for delegable capabilities (OQ-7). | — | Unverified. |

## Remaining verification items

The independent review reported by the repository owner covers the
official sources for RFC 8259, RFC 7493, RFC 8785 (§3.1, §3.2.3), RFC 7515
(§4.1.11), RFC 3339, RFC 4648, RFC 3986, JSON Schema 2020-12, W3C PROV-DM,
SLSA v1.2 (specification and provenance), OpenID Connect Core and
FIPS 180-4. These items remain:

1. JSON Schema: offline `$id` resolution behavior in the validators chosen
   for the reference tooling. This is an implementation question, not a
   specification one.
2. Standards marked out of scope (CBOR, CDDL, COSE, Protocol Buffers, W3C
   Verifiable Credentials) are unverified. They are not used in v0.1, so
   they do not block acceptance.

None of these verifications implies implementation conformance with any
standard (D5.4).
