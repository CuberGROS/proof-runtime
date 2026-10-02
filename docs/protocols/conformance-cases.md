# Proposed Conformance Cases: Shared Foundations

> **Status: Proposed**, as part of
> [ADR 0002](../adr/0002-shared-protocol-foundations.md). These are prose
> test-case descriptions for the shared extension, version and integrity
> rules. They are not test vectors, schemas or code. Concrete vectors can
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
| X-09 | Critical list contains duplicate identifiers or non-ASCII identifiers | Invalid; reject | Invalid; reject | D1.2, D3.4 |
| X-10 | A consumer that understands `E-INFO` attempts to use it as input to a decision | — | Non-conformant consumer; a test harness must detect that its decision differs from X-07's | D3.3 |

**X-03 notes.** A missing critical marker on a restrictive extension is the
case the extension model cannot fully prevent: an unaware consumer has no
way to know that the extension mattered. Required mitigations:

- producer-side validators MUST fail this case before emission,
- aware consumers reject it,
- future authenticity (D4.7) binds the producer to its own error.

## Versions (D2)

| ID | Case | Expected result | Rule |
|---|---|---|---|
| V-01 | Major in type identifier differs from `MAJOR` in the declared protocol version | Invalid; reject | D2.2 |
| V-02 | Document major version not implemented by the consumer | Reject; no best-effort parsing | D2.3 |
| V-03 | Protocol `0.x`: document `0.3`, consumer implements `0.2` only, no declared compatibility | Reject | D2.6 |
| V-04 | Protocol `0.x`: document `0.3`, consumer implements `0.2`, and the specification explicitly declares `0.2` and `0.3` compatible | Process | D2.6 |
| V-05 | Protocol `≥1.0`: document `1.4`, consumer implements `1.2` | Process; unrecognized core members are ignored, and they are decision-neutral by D2.5 | D2.4, D2.5 |
| V-06 | Protocol `≥1.0`: a minor revision introduces a core member that affects a decision | Specification defect: must be a major version or a critical extension | D2.5 |
| V-07 | Malformed protocol version (not `MAJOR.MINOR`) | Invalid; reject | D2.2 |

## Integrity (D4)

| ID | Case | Expected result | Rule |
|---|---|---|---|
| I-01 | Document received together with a stated digest; the consumer recomputes, and the digests match | The digest is accepted for this document | D4.5 |
| I-02 | Stated digest does not match the recomputed digest | The digest is not accepted; any claim relying on it is unverifiable | D4.5 |
| I-03 | Relay parses and re-serializes with different whitespace and member order, preserving all members losslessly | Bytes differ; content digest is identical to the original | D4.2, D4.5 |
| I-04 | Relay's parser rounds an extension number it cannot represent exactly, then re-serializes | Content digest differs. The relay was required to forward the original bytes or treat the output as a transformation. Presenting it with the original's digest is non-conformant. | D4.5 |
| I-05 | A critical marker is removed after the original digest was recorded independently | Recomputed digest does not match the recorded digest; the document is unverifiable | D3.7, D4.5 |
| I-06 | Migration or redaction produces a new document that is presented with the original's digest | Non-conformant. The new document needs its own digest and a reference to the original. | D2.7, D4.5 |
| I-07 | Reference carries only digest algorithms the consumer does not accept | Unverifiable; never treated as matching | D4.4 |
| I-08 | A v0.1 document or implementation claims to be signed or DSSE-conformant | Non-conformant claim | D4.7 |
