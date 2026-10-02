# Conformance Cases: Shared Foundations

> **Status: Accepted**, as part of
> [ADR 0002](../adr/0002-shared-protocol-foundations.md) (accepted
> 2026-10-02). These are prose test-case descriptions for the shared
> structure, extension, version and integrity rules. A case marked
> "Optional (MAY)" describes permitted behavior, not a mandatory acceptance
> test. They are not test vectors, schemas or code. Concrete vectors can
> only be written once protocol fields exist, so they belong to the
> protocol specifications.

## Conventions

- **Consumer kinds:**
  - **Aware:** understands the extension in question, at the listed major
    version.
  - **Unaware:** does not understand it.
- **Expected results:**
  - **Reject:** the consumer refuses the document for every
    security-relevant decision, records the reason, and performs no partial
    processing.
  - **Invalid:** a producer-side validator or an aware consumer reports
    that the document violates ADR 0002.
  - **Process:** the consumer handles the document normally.
- **Decision.** "Decision" means the security-relevant decision, as defined
  in [ADR 0002 § Terms used in this decision](../adr/0002-shared-protocol-foundations.md#terms-used-in-this-decision).
- **Placeholder names.** `E-RESTRICT` names an extension specified as
  critical because it **restricts** a decision, for example by narrowing a
  capability. `E-PERMIT` names an extension specified as critical because it
  **relaxes** a decision, for example by allowing an additional action.
  `E-INFO` names an extension specified as non-critical, carrying only
  decision-neutral annotation.

## Structure and numbers (D1)

| ID | Case | Expected result | Rule |
|---|---|---|---|
| S-01 | Document is valid JSON under the profile but fails its protocol's normative JSON Schema, for example by missing a required member | Reject | D1.3, C2 |
| S-02 | Number token `9007199254740993` (integer spelling, above 2^53 − 1), in core or extension data | Invalid; reject. Detected lexically, before any binary64 conversion, which would round it to `9007199254740992`. | D1.2 |
| S-03 | Number token `9007199254740993.0` (integral value with a fraction part) | Invalid; reject (fraction spelling) | D1.2 |
| S-04 | Number token `9.007199254740993e15` (integral value with an exponent) | Invalid; reject (exponent spelling) | D1.2 |
| S-05 | Number token `1e400` (overflows binary64) | Invalid; reject (exponent spelling) | D1.2 |
| S-06 | Number token `0.5` | Invalid; reject. A fraction must be a string in its owning specification's format. | D1.2 |
| S-07 | Number token `1e0` (integral value 1 with an exponent) | Invalid; reject (exponent spelling) | D1.2 |
| S-08 | Number token `-0` | Invalid; reject (negative zero) | D1.2 |
| S-09 | Number tokens `9007199254740991`, `-9007199254740991` and `0` | Accepted; each is a valid integer spelling within the safe range | D1.2 |
| S-10 | An exact-precision quantity (for example, a monetary amount) is encoded as a JSON number in extension data | Non-conformant. Integral amounts within range parse but violate the string requirement; fractional amounts are already rejected (S-06). | D1.2 |

## Extensions (D3)

| ID | Case | Unaware consumer | Aware consumer | Rule |
|---|---|---|---|---|
| X-01 | `E-RESTRICT` present and listed as critical | Reject | Process; the restriction is applied | D3.4, D3.5 |
| X-02 | `E-PERMIT` present and listed as critical | Reject; the permission is never honored | Process; the permission is evaluated by CONTROL as specified | D3.4, D3.5 |
| X-03 | `E-RESTRICT` present but **not** listed as critical (missing critical marker) | Cannot detect the error; ignores the extension. This is the stated residual risk; see X-03 notes. | Invalid; reject | D3.6 |
| X-04 | `E-PERMIT` present but **not** listed as critical (missing critical marker) | Ignores the extension; the permission is never granted | Invalid; reject | D3.3, D3.6 |
| X-05 | Critical list names an identifier that is not present in the extensions element | Invalid; reject | Invalid; reject | D3.4 |
| X-06 | Critical extension listed at a major version the consumer does not support (for example, it supports `/v1`, the document carries `/v2`) | Reject (`/v2` is not understood) | Reject (`/v2` is not understood) | D3.2, D3.4 |
| X-07 | `E-INFO` present, not listed as critical | Ignores it. The decision is identical to the same document without `E-INFO`. | Uses it only for non-decision purposes. The decision is identical to the same document without `E-INFO`. | D3.3 |
| X-08 | Extension data appears as a member outside the designated extensions element | Invalid; reject | Invalid; reject | D3.1 |
| X-09 | Critical list contains the same identifier twice, or contains a non-ASCII identifier | Invalid; reject | Invalid; reject | D1.2, D3.4 (uniqueness and ASCII) |
| X-10 | A consumer that understands `E-INFO` attempts to use it as input to a decision | — | Non-conformant consumer; a test harness must detect that its decision differs from X-07's | D3.3 |
| X-11 | An author adds a new restricting member to `E-RESTRICT` without changing its identifier | — | An older aware consumer would treat the identifier as understood and ignore the new member. Specification defect: the change requires a new identifier. Producer-side validation must fail. | D3.2 |

**X-03 notes.** A missing critical marker on a restrictive extension is the
case the extension model cannot fully prevent: an unaware consumer has no
way to know that the extension mattered. Required mitigations:

- producer-side validators MUST fail this case before emission,
- aware consumers reject it.

Future signatures would only attribute the error to its producer. They
would not let an unaware consumer reject the document, so they are not a
mitigation. For unaware consumers, this risk is unmitigated (D3.6).

## Versions (D2)

| ID | Case | Expected result | Rule |
|---|---|---|---|
| V-01 | Major in type identifier differs from `MAJOR` in the declared protocol version | Invalid; reject | D2.2 |
| V-02 | Document major version not implemented by the consumer | Reject; no best-effort parsing | D2.3 |
| V-03 | Protocol `0.x`: document `0.3`, consumer implements `0.2` only | Reject | D2.6 |
| V-04 | Protocol `0.x`: a specification declares `0.2` and `0.3` compatible | Specification defect: `0.x` compatibility must not be declared. A consumer that implements only `0.2` rejects a `0.3` document regardless. | D2.6 |
| V-05 | Protocol `≥1.0`: document `1.4`, consumer implements `1.2` | **Optional (MAY).** Process and reject both conform. If the consumer processes the document, it ignores unrecognized core members, which are decision-neutral by D2.5, and handles unknown values of open members as originally specified. Not a mandatory acceptance test. | D2.4, D2.5 |
| V-06 | Protocol `≥1.0`: a minor revision introduces a core member that affects a decision | Specification defect: must be a major version or a critical extension | D2.5 |
| V-07 | Malformed protocol version (not `MAJOR.MINOR`) | Invalid; reject | D2.2 |
| V-08 | Protocol `≥1.0`: a minor revision adds a value to a member whose original definition did not declare its value space open | Specification defect: requires a new major version | D2.4 |
| V-09 | Protocol `≥1.0`: document `1.4` uses a value unknown to a `1.2` consumer, in a member declared open with "reject unknown values" handling | Reject, as the original definition specifies | D2.4 |

## Integrity (D4)

| ID | Case | Expected result | Rule |
|---|---|---|---|
| I-01 | Document received together with a stated digest; the consumer recomputes, and the digests match | The digest is accepted for this document | D4.5 |
| I-02 | Stated digest does not match the recomputed digest | The digest is not accepted; any claim relying on it is unverifiable | D4.5 |
| I-03 | Relay parses and re-serializes with different whitespace and member order, preserving all members losslessly | Bytes differ; content digest is identical to the original | D4.2, D4.5 |
| I-04 | Relay's parser rounds an extension number it cannot represent exactly, then re-serializes | Content digest differs. The relay was required to forward the original bytes or treat the output as a transformation. Presenting it with the original's digest is non-conformant. | D4.5 |
| I-05 | A critical marker is removed after the original digest was recorded independently | Recomputed digest does not match the recorded digest; the document is unverifiable | D3.7, D4.5 |
| I-06 | Migration or redaction produces a new document that is presented with the original's digest | Non-conformant. The new document needs its own digest and a reference to the original. | D2.7, D4.5 |
| I-07 | Reference carries only digest algorithms the consumer does not accept. Because `sha256` is mandatory and always accepted in v0.1, this means `sha256` is absent. | Invalid reference; reject (same outcome as I-12) | D4.4 |
| I-08 | A v0.1 document or implementation claims to be signed, to have established authenticity, or to conform to DSSE or another signing envelope | Non-conformant claim | D4.6 |
| I-09 | A digest set holds `sha256` of content A and `sha512` of content B (a producer error) | Producer violates D4.4. A consumer that verifies both entries matches neither A nor B, and rejects. A consumer that verifies only `sha256` resolves to A. **No conformant consumer resolves to B**, because the `sha256` anchor must always match. | D4.4 |
| I-10 | A digest set holds `sha256` and `sha512`, both computed over the candidate content | Match | D4.4 |
| I-12 | A v0.1 digest reference contains only `sha512` (no `sha256`) | Invalid reference; reject | D4.4 |
| I-13 | A reference's target has been deleted or tombstoned under the retention policy | Unverifiable. It is never presented as verified. Deletion is not an in-place edit. | D2.7 |
| I-11 | Two serializations differ only in member order and different JSON string spellings of the same parsed content (`"\u0061"` versus `"a"`) | Same document content digest. This is canonical-content integrity, not byte integrity; byte equality is not claimed. | D4.1, D4.2 |
