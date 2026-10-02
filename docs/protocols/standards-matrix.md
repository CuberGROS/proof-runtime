# Standards Compatibility and Reuse Matrix

> **Status: Proposed**, as part of
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

This review ran with restricted network access. The RFC Editor, IETF
Datatracker, W3C, `openid.net`, `slsa.dev`, `modelcontextprotocol.io` and
`a2a-protocol.org` were blocked by the review environment's egress proxy.
Each claim below carries one of these levels:

- **Verified (primary).** Read in this review from the official
  specification or the standard's official source repository.
- **Corroborated.** The official page was located by web search, and the
  search engine's excerpt of it supports the claim. The full text was not
  retrieved. These claims must be re-checked against the full text before
  ADR 0002 is accepted.
- **Secondary.** Supported only by third-party sources.
- **Unverified.** Not checked in this review; stated from prior knowledge.

## Matrix

### Encoding, schema and integrity building blocks

| Standard (official source) | Disposition | What Proof Runtime takes | Caveats | Verification |
|---|---|---|---|---|
| JSON — [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259) | Reuse | Interchange encoding (D1.1). | — | Unverified. |
| I-JSON — [RFC 7493](https://www.rfc-editor.org/rfc/rfc7493) | Reuse | Profile base (D1.2). | Proof Runtime is stricter: integer-only core numbers, no `null` for absence, ASCII identifiers. | **Corroborated:** "Objects in I-JSON messages MUST NOT have members with duplicate names"; integers outside [−(2^53)+1, (2^53)−1] cannot be expected to be treated as exact. **Unverified:** the RFC's exact wording on surrogates and its protocol-design recommendations. |
| JSON Schema 2020-12 — [Validation §7.2](https://json-schema.org/draft/2020-12/json-schema-validation); [version status](https://json-schema.org/specification) | Reuse | Structural validation (D1.3). | `format` is not relied on for security checks. Schemas resolve offline. Validity is necessary, not sufficient. | **Verified (primary):** 2020-12 is the current version. Under the Format-Annotation vocabulary, `format` "MUST be collected as an annotation", and optional assertion "MUST be disabled by default" (§7.2.1). **Unverified:** offline `$id` resolution behavior in specific validators. |
| JCS — [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785); [author's repository](https://github.com/cyberphone/json-canonicalization) | Reuse | Canonical form for content digests (D4.2). | Informational, Independent Submission: the RFC Editor makes no statement about its value for implementation. No Unicode normalization. ECMAScript-style number serialization. | **Corroborated:** Informational, Independent Submission, June 2020; builds on ECMAScript serialization, the I-JSON subset and deterministic property sorting. **Verified (primary, author's repository):** no Unicode normalization; implementations in Rust, JavaScript, Java, Go, .NET, Python and others. **Unverified:** the exact property-sorting rule (UTF-16 code units). |
| SHA-256 — FIPS 180-4 | Reuse | Mandatory-to-support digest (D4.4). | Algorithm agility through `DigestSet`. | Unverified. |
| in-toto `DigestSet` — [digest_set.md](https://github.com/in-toto/attestation/blob/main/spec/v1/digest_set.md) | Reuse (shape only) | Digest representation (D4.4). | Proof Runtime adds: a reference with no acceptable algorithm is unverifiable. | **Verified (primary):** algorithm name → lowercase hex; consumers "MUST only accept algorithms that they consider secure and MUST ignore unrecognized or unaccepted algorithms"; `sha256` recommended. |
| DSSE — [protocol.md](https://github.com/secure-systems-lab/dsse/blob/master/protocol.md) | **Reserved** | Preferred future signing envelope (D4.7). | Not normative in v0.1. No signing infrastructure exists. No DSSE conformance is claimed. | **Verified (primary):** signs `PAE(type, body)` = `"DSSEv1" SP LEN(type) SP type SP LEN(body) SP body`; `payloadType` identifies interpretation. |
| RFC 3339, RFC 4648, RFC 3986 — [3339](https://www.rfc-editor.org/rfc/rfc3339), [4648](https://www.rfc-editor.org/rfc/rfc4648), [3986](https://www.rfc-editor.org/rfc/rfc3986) | Reuse | Timestamps, base64url, URI syntax (D1.2, D3.2). | Timestamps are producer claims (OQ-28). | Unverified. |
| JWS — [RFC 7515 §4.1.11](https://www.rfc-editor.org/rfc/rfc7515#section-4.1.11) | Interoperate | Its `crit` header is the model for D3.4. JWS over JCS is a possible later alternative envelope, for example for parity with A2A Agent Cards. | Not the designated envelope. | **Corroborated:** `crit` lists extensions that "MUST be understood and processed"; if any is not understood and supported by the recipient, the JWS is invalid. Several 2026 library vulnerabilities (for example CVE-2026-32597 in PyJWT) concern implementations that failed to enforce this (secondary). |
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
| W3C PROV-DM — [W3C TR](https://www.w3.org/TR/prov-dm/) | Align | The entity / activity / agent distinction, for Evidence Receipt provenance vocabulary (OQ-17, OQ-21). | No RDF or PROV-O serialization in v0.1. | **Corroborated:** W3C Recommendation dated 30 April 2013. Entity: "a physical, digital, conceptual, or other kind of thing with some fixed aspects". Activity: "something that occurs over a period of time and acts upon or with entities". Agent: "something that bears some form of responsibility for an activity taking place, for the existence of an entity, or for another agent's activity". |
| in-toto Attestation Framework v1 — [README](https://github.com/in-toto/attestation/blob/main/spec/v1/README.md), [Statement](https://github.com/in-toto/attestation/blob/main/spec/v1/statement.md) | Interoperate | A future exporter may present Evidence Receipts as in-toto Statements. D2.5 and D3.5 extend its monotonic principle to both directions. | Subjects are digest-identified artifacts; receipts also describe actions and decisions, so a mapping must be specified. | **Verified (primary):** `_type` `https://in-toto.io/Statement/v1`; each subject "MUST have `digest` set"; "Consumers MUST ignore unrecognized fields unless otherwise noted"; monotonic principle. |
| SLSA — [specification index](https://slsa.dev/spec/); [v1.2 announcement](https://slsa.dev/blog/2025/11/announce-slsa-v1.2) | Out of scope (core); Interoperate (software-engineering extensions) | SLSA provenance may appear as digest-referenced evidence in software-engineering extensions. | Software supply-chain scope. It must not shape the industry-neutral core (D3.8). | **Corroborated:** v1.2 is the current published version (announced November 2025); it adds a Source Track and is described as backwards compatible with v1.1. The official GitHub repository has no tagged releases. **Unverified:** the provenance `predicateType` URI. |

### Identity

| Standard (official source) | Disposition | What Proof Runtime takes | Caveats | Verification |
|---|---|---|---|---|
| OpenID Connect Core 1.0 — [specification](https://openid.net/specs/openid-connect-core-1_0.html) | Interoperate (deferred to OQ-5) | Candidate source of human and service principal identity. | OIDC authenticates an End-User; it does not authorize actions. An ID Token is not a capability (C7). | **Corroborated:** "OpenID Connect Core 1.0 incorporating errata set 2", 15 December 2023; "a simple identity layer on top of the OAuth 2.0 protocol" that lets clients "verify the identity of the End-User based on the authentication performed by an Authorization Server". |
| SPIFFE — [SPIFFE-ID.md](https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE-ID.md) | Interoperate (deferred to OQ-5) | Candidate workload identity for runtime and host components. | Workload identity is not authorization. | **Verified (primary):** `spiffe://trust-domain/path`; X.509-SVID, JWT-SVID and WIT-SVID formats. |
| W3C Verifiable Credentials | Out of scope | Possible future mechanism for delegable capabilities (OQ-7). | — | Unverified. |

## Items to re-check before acceptance

These claims are corroborated or unverified, and must be checked against
the full primary text before ADR 0002 is accepted:

1. RFC 7493: exact wording on surrogates and noncharacters, and its
   protocol-design recommendations.
2. RFC 8785: the full canonicalization rules, including the property-sorting
   rule.
3. RFC 7515 §4.1.11: exact `crit` text.
4. W3C PROV-DM: the full text of its Recommendation status and definitions.
5. SLSA: the current version (v1.2 or later) and the provenance
   `predicateType` URI.
6. OpenID Connect Core: the current errata set.
7. JSON Schema: offline `$id` resolution in the validators chosen for the
   reference tooling.
8. RFC 8259, RFC 3339, RFC 4648, RFC 3986, FIPS 180-4: not checked in this
   review. These are long-established standards, but they are listed for
   completeness.
