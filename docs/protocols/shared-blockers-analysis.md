# Shared Blocker Decisions: Analysis and Recommendations

> **Status: Proposed**, supporting
> [ADR 0003](../adr/0003-shared-blocker-decisions.md) (status: **Proposed**).
> This document explains the reasoning behind the proposed decisions D6 to
> D9. The proposed decisions themselves are stated only in the ADR. If the
> two disagree, the ADR governs, and the disagreement is a defect. Nothing
> here resolves an open question: each question stays open until its
> decision in ADR 0003 is explicitly accepted.

This document covers the four questions that block field-level
specification of several or all core protocols
([dependency map](dependency-map.md)):

| Question | Topic | Proposed decision |
|---|---|---|
| [OQ-26](#oq-26-identifier-namespace) | Identifier namespace | D6 |
| [OQ-27](#oq-27-protocol-resource-limits) | Protocol resource limits | D7 |
| [OQ-5](#oq-5-identity-model) | Identity model | D8 |
| [OQ-29](#oq-29-control-decision-records) | CONTROL decision records | D9 |

They are analyzed in dependency order. OQ-26 and OQ-27 depend on nothing
else. OQ-5 uses the identifier rules of OQ-26 and the limits of OQ-27.
OQ-29 uses all three.

Each section covers:

1. requirements,
2. alternatives,
3. security implications,
4. the recommendation,
5. unresolved dependencies,
6. proposed conformance cases (prose only, not normative until accepted),
7. residual risks.

Every section works within the frozen boundaries of
[ADR 0001](../adr/0001-architecture-freeze.md) and the accepted decisions
of [ADR 0002](../adr/0002-shared-protocol-foundations.md). None of them
adds a plane component, a core protocol or an invariant. Protocol fields
and JSON Schemas are out of scope.

Citations, and the extent to which each could be checked in this session,
are listed in [Sources and verification](#sources-and-verification).

---

## OQ-26: Identifier namespace

> ADR 0002: "Which URI authority will host Proof Runtime document type
> identifiers and the reserved core extension namespace? It must be durable
> and under the project's control. Resolving this needs an owner decision
> about a domain or other namespace."

### Requirements

- **R26.1 Absolute URI.** Type and extension identifiers are absolute URIs
  (D2.2, D3.2). They are ASCII and compared as exact, case-sensitive
  strings with no normalization (D1.2).
- **R26.2 Durable.** An identifier, once published, must keep its meaning
  for as long as documents that carry it are retained. Retention can last
  years (OQ-19).
- **R26.3 Under the project's control.** Only the project can mint
  identifiers in the core namespace (D3.2 reserves it for protocol
  specifications).
- **R26.4 Carries the major version.** The type identifier encodes the
  protocol's major version using the D2.2 component syntax. Its position in
  the URI is left to OQ-26 (D2.2).
- **R26.5 Never fetched.** Schemas are resolved offline by `$id`, and
  validators never fetch `$ref` targets (D1.3). Correct processing must not
  depend on any identifier being resolvable.
- **R26.6 One spelling.** Because comparison is exact (D1.2), every
  identifier must have exactly one spelling, or two producers will emit
  different strings for the same intended identifier.

### Alternatives

| Option | Form | Durability | Project control | Cost and process | Assessment |
|---|---|---|---|---|---|
| **A. Project-controlled DNS domain** | `https://{domain}/...` | As long as the domain is renewed. A lapse lets a third party register the domain and mint look-alike identifiers. | Full, once registered. | Annual registration fee. Needs a registrant and renewal discipline. | The conventional choice for JSON Schema `$id` values, in-toto predicate types and A2A extension URIs. Humans can look up documentation. |
| **B. w3id.org permanent identifier** | `https://w3id.org/{project}/...` | The service is run by a consortium of organizations, and identifiers are "intended to be around for as long as the Web is around" (w3id README). The redirect target can move if hosting changes. | Partial. The project controls the redirect target, through pull requests to `perma-id/w3id.org` that the service's maintainers review and merge. Administrators "may deny requests for identifiers that are too generic". | No fee. A pull request to a third-party repository. | Durable and free. But the URI authority belongs to a third party, so R26.3 is met only in part. |
| **C. `tag:` URI (RFC 4151)** | `tag:{domain-or-email},{YYYY-MM-DD}:...` | Very high. The date fixes ownership at minting time, so a later owner of the domain cannot mint the same tags. | Requires control of a domain or email address on the minting date only. | No registration. No resolution mechanism. | Best resistance to domain lapse, and naturally never fetched (R26.5). Unfamiliar to most implementers. Does not give humans a documentation link. Using a personal email address would tie the namespace to an individual. |
| **D. Formal URN namespace (RFC 8141)** | `urn:{nid}:...` | Very high once registered. | High, after registration. | IANA registration with expert review: slow, and heavy for a pre-alpha project. | Disproportionate now. Could be adopted later through a new ADR. |
| **E. GitHub-hosted URL** | `https://github.com/CuberGROS/...` or `https://cubergros.github.io/...` | Tied to the account name and the repository's location. After a rename or transfer, the old name can be claimed by someone else. | GitHub controls the authority. The project controls only a path under an account name. | Free and immediate. | Fails R26.2 and R26.3. Not recommended for identifiers, though suitable for hosting human-readable documentation. |
| **F. `urn:uuid:` (random)** | `urn:uuid:...` | Unique forever. | None. Anyone can mint any UUID, so ownership of a namespace cannot be expressed. | Free. | Cannot express a reserved core namespace (R26.3). Opaque to humans. Rejected for the core namespace. |

### Security implications

- **Identifiers are names, not locators.** If a consumer fetched anything
  from an identifier, then whoever controls the identifier's host would
  control the consumer's behavior. D1.3 already forbids fetching schemas.
  The proposed decision extends this to every identifier: no conformant
  processing step may dereference a type identifier, an extension
  identifier or a schema `$id`. With that rule in place, the technical
  impact of losing a domain under option A is limited to human confusion.
  The attacker can publish misleading documentation or "specifications", but
  no conformant consumer's decision changes.
- **Namespace lapse and squatting.** Under option A, a lapsed domain lets a
  third party mint new identifiers that look official, for example a fake
  "v2" type identifier or a core-looking extension.
  - Consumers recognize core types only by exact match against the
    identifiers bundled with them. An unknown major version is rejected
    (D2.3), and an unknown critical extension fails closed (D3.4).
  - The residual risk is social: an implementer could be persuaded to bundle
    a malicious "specification". Operational safeguards reduce it (D6.6).
    Option C removes it, at the cost of discoverability.
- **Prefix matching.** A consumer that treats any identifier under the core
  prefix as trusted or "known" can be fooled by anyone who writes such a
  string; writing a URI requires no control of its authority. Namespace
  membership must confer nothing. Recognition is only by exact match.
- **Spelling divergence.** RFC 3986 treats `HTTPS://Example.ORG/a` and
  `https://example.org/a` as equivalent, but exact comparison (D1.2) does
  not. If spelling were unconstrained:
  - A critical extension spelled differently from the spelling a consumer
    knows looks unknown, and the document is rejected. This fails closed,
    but it harms interoperability.
  - A non-critical extension spelled differently is ignored. This is safe,
    because non-critical extensions are decision-neutral (D3.3).

  A single canonical spelling for core identifiers removes the first
  problem for core types. It is also recommended for extension identifiers.
- **Namespace is not identity.** The authority component of a type
  identifier says which specification a document claims to follow. It says
  nothing about who produced the document. Anyone can produce a document
  carrying a core type identifier. Producer identity is OQ-5, and
  authenticity is the OQ-4 residual.

### Recommendation

**Option A, a project-controlled DNS domain with `https` identifiers,
combined with the rule that identifiers are never dereferenced and with
operational safeguards against lapse.**

Why A rather than B or C:

- Under the never-dereference rule (D6.2), the main technical weakness of A
  (a lapsed domain) cannot change any conformant consumer's decision. What
  remains is a social-engineering risk, and the safeguards in D6.6 reduce
  it.
- A meets R26.3 fully. B meets it only through a third party's review
  process.
- `https` identifiers match established practice for JSON Schema `$id`,
  in-toto predicate types and A2A extensions. This lowers the chance that
  implementers mis-handle them, and lets humans find documentation.

Ranked fallbacks, in case the owner prefers not to hold a domain:

1. **Option C (`tag:`)**, if resistance to lapse matters more than
   discoverability. It still requires a domain, or a role email address
   (not a personal one), controlled on the minting date.
2. **Option B (w3id.org)**, if no domain or role email is available. Under
   B the project depends on a third party's governance, so R26.3 holds only
   in part.

Options D, E and F are not recommended.

**Owner decision required.** The project cannot pick the authority on the
owner's behalf. The owner must:

1. choose option A, B or C,
2. under A, choose and register the domain, decide who the registrant is
   (preferably an organization or role account rather than an
   individual), and commit to renewal,
3. under C, choose the tagging entity and the minting date,
4. under B, choose the w3id.org top-level name and submit the redirect
   request,
5. confirm or change the identifier template and the minting rule below.

**Proposed identifier template** (independent of the option chosen, once
`{authority}` is fixed):

```text
core type identifier:      {root}/{protocol}/v{MAJOR}
core extension identifier: {root}/ext/{name}/v{MAJOR}
schema $id (SHOULD):       {root}/{protocol}/v{MAJOR}/schema/{MAJOR}.{MINOR}
```

- `{root}` is the core namespace root, for example `https://{domain}/{base}`
  under option A. It is fixed by the owner decision.
- `{protocol}` is one of `task-capsule`, `action-ir`,
  `capability-manifest` and `evidence-receipt`. Only the four frozen core
  protocols can appear here (ADR 0001 §2).
- `{MAJOR}` and `{MINOR}` use the D2.2 component syntax. `v1` is valid;
  `v01` is not.
- D9 may need a form segment for the decision-record form of the Evidence
  Receipt protocol: `{root}/evidence-receipt/{form}/v{MAJOR}`. Whether a
  type identifier or a core member distinguishes forms is left to the
  Evidence Receipt specification (D9.2).

**Placeholder for drafts.** Until the owner decides, drafts use the
reserved, non-resolvable authority `example.invalid` (RFC 2606 and RFC 6761
reserve `.invalid`). A draft specification can then be reviewed with a
syntactically valid identifier that can never collide with a real one. No
published specification version may contain it.

### Unresolved dependencies

- The owner decision above. Until it is made, no core type identifier is
  final, and no protocol specification can be published.
- Interaction with D9: the form segment, if the Evidence Receipt
  specification uses one.

### Proposed conformance cases

| # | Case | Expected result |
|---|---|---|
| N-01 | Type identifier equals a bundled core identifier except for an uppercase host | Unknown type. Reject. |
| N-02 | Type identifier equals a bundled core identifier with a trailing `/` | Unknown type. Reject. |
| N-03 | Type identifier is under the core prefix but is not a bundled identifier (for example, an unpublished `v7`) | Unknown type. Reject. It is never treated as "core" because of its prefix. |
| N-04 | Consumer encounters an identifier whose host does not resolve or is offline | No change in behavior. Identifiers are never dereferenced. |
| N-05 | Producer emits a core identifier containing a percent-encoded character, a query or a fragment | Invalid (producer-side) under D6.3. |
| N-06 | A published specification contains the `example.invalid` placeholder | Invalid publication under D6.7. |

### Residual risks

- Under option A, social-engineering risk if the domain lapses (see above).
- Extension identifiers are owned by their authors, and the project cannot
  enforce canonical spelling for them. Misspelling fails closed for
  critical extensions (D3.4).
- The template is a proposal. Protocol specifications may find they need
  additional reserved paths. Each one needs a decision under D6.5.

---

## OQ-27: Protocol resource limits

> ADR 0002: "What maximum document size, nesting depth, string length and
> array length does each protocol permit?" D1.4 requires every protocol
> specification to define at least maximum document size and maximum
> nesting depth, and consumers to reject documents that exceed the limits
> "before any further processing".

### Requirements

- **R27.1 Bounded work before trust.** Every document is untrusted input
  until validated. The cost of rejecting a hostile document must be bounded
  by limits checked before schema validation, canonicalization, hashing,
  reference resolution or any security-relevant use (D1.4, C2).
- **R27.2 Checkable before the protocol is known.** JSON member order
  carries no meaning, so a consumer cannot know a document's type or
  version until it has tokenized the whole document. At least one set of
  limits must therefore be independent of the protocol.
- **R27.3 Streaming-checkable.** Every limit must be checkable in one pass
  over the bytes, in memory bounded by the limits themselves, without
  building a full in-memory tree first.
- **R27.4 Same answer in every language.** Every conformant consumer, in
  any language, must accept or reject the same document. Every measure must
  therefore be defined exactly. In particular, string length cannot be left
  to a language's native notion of "length" (UTF-16 code units in
  JavaScript, Java and C#; code points in Python; bytes in Rust and Go).
- **R27.5 Inside common parser defaults.** No valid document should be
  rejected by a mainstream JSON parser using its default limits. A limit
  above some parser's default would make conformance depend on parser
  configuration.
- **R27.6 Room for real documents.** Limits must not prevent legitimate
  content. Bulk content (logs, files, binaries) is referenced by digest, not
  embedded (D1.2), so documents are structured metadata.

### Evidence for the chosen values

Default nesting limits of widely used JSON parsers, read from their source
code in this session (see [sources](#sources-and-verification)):

| Parser | Default limit | Source |
|---|---|---|
| .NET `System.Text.Json` | Depth 64 | `JsonReaderOptions.cs`: `DefaultMaxDepth = 64` |
| Rust `serde_json` | Recursion 128 | `de.rs`: `remaining_depth: 128` |
| Ruby `json` | Nesting 100 | `parser.c`: `config->max_nesting = 100` |
| Java Jackson 2.18 | Depth 1000; string 20,000,000 characters; number 1,000 characters; name 50,000 characters | `StreamReadConstraints.java` |
| Go `encoding/json` | Depth 10000 | `scanner.go`: `maxNestingDepth = 10000` |
| CPython | Recursion limit 1000, which bounds recursive parsing | `pycore_ceval.h`: `Py_DEFAULT_RECURSION_LIMIT 1000` |

RFC 8259 §9 explicitly allows implementations to limit text size, nesting
depth, number range and precision, and string length and contents
**(corroborated)**. So limits are expected, but there is no
interoperable default. That is why the protocols must set their own.

### Alternatives

| Option | Description | Assessment |
|---|---|---|
| **L1. Per-protocol limits only** | Each specification sets its own limits, as D1.4 minimally requires. | Fails R27.2. A consumer cannot apply limits to a document whose type it does not yet know, so the first pass is unbounded. |
| **L2. One shared ceiling only** | One set of limits for every protocol. | Meets R27.2, but cannot express a tighter bound for a protocol that needs one. On its own, it also fails to meet D1.4's per-protocol requirement. |
| **L3. Shared ceiling plus per-protocol limits at or below it** | A protocol-independent ceiling is checked first. Each specification then states its own limits, each at most the ceiling, and they are checked once the type is known. | Meets every requirement. **Recommended.** |
| **L4. Producer-declared limits** | A document states its own limits. | Circular: a hostile producer declares large limits. Rejected. |
| **L5. Consumer-configured limits only** | Each deployment chooses. | Fails R27.4. Two consumers would disagree on the same document. Rejected as the normative mechanism. Local stricter refusal is still allowed (D7.6). |

### Security implications

- **Memory exhaustion.** A parsed tree takes much more memory than its text
  in most languages. The size ceiling bounds this for every consumer, and
  the total-value ceiling bounds it further for documents built from many
  tiny values. JCS (RFC 8785) sorts members, so the whole data model must be
  in memory to canonicalize it. Canonicalization cannot be streamed, which
  makes the size ceiling the real bound on digest computation.
- **Stack exhaustion.** Recursive parsers, canonicalizers and schema
  validators (which may recurse further through `$ref`) can overflow the
  stack on deep nesting. A depth ceiling well below the smallest
  mainstream default (64) leaves headroom for those extra frames.
- **Algorithmic complexity.**
  - Uniqueness checks are often quadratic in naive implementations. They
    occur in JSON Schema `uniqueItems`, in duplicate detection in the
    critical list (D3.4) and in any set semantics a specification defines.
    The array ceiling bounds them.
  - Duplicate-member detection (D1.2) and JCS sorting are bounded by the
    per-object member ceiling. That ceiling also keeps worst-case hash
    collision behavior acceptable for hash tables without randomized seeds.
  - Converting very long number tokens is superlinear in some libraries.
    The D1.2 integer profile already fixes a maximum token length of 17
    bytes (`-9007199254740991`), so consumers can reject longer tokens
    lexically, without converting them.
- **Hash amplification.** Each accepted digest algorithm in a digest set
  costs one full hash of the referenced content (D4.4, all-of-accepted
  matching). Limiting entries per digest set bounds that cost per
  reference.
- **Log amplification and injection.** Bounded strings limit how much
  attacker-controlled text a rejection or audit log can be made to carry.
- **Parser differentials.** If two consumers enforce different measures,
  for example UTF-16 length versus UTF-8 length, an attacker can craft a
  document that one accepts and the other rejects. Exact, byte-based
  definitions remove this (R27.4).
- **Availability only, never authority.** Exceeding a limit always leads to
  rejection. No limit decision ever leads to more permissive processing, so
  limits cannot weaken an authorization decision.

### Recommendation

**Option L3**, with the following ceiling values. Each protocol
specification states its own limits for every measure, at or below the
ceiling. If a specification does not state a tighter value, the ceiling
applies.

| # | Measure (defined exactly in D7.2) | Ceiling | Justification |
|---|---|---|---|
| 1 | Document size: bytes of the JSON text | **1,048,576** (1 MiB) | Documents carry metadata only. Bulk content is referenced by digest (D1.2). 1 MiB leaves ample room for large manifests or receipts while bounding the whole-document memory that JCS needs to a size every platform handles comfortably. |
| 2 | Nesting depth | **32** | Half the smallest mainstream default (64, `System.Text.Json`), leaving stack headroom for recursive validation and canonicalization (R27.5). Core structures are not expected to need more than about ten levels. Extensions sit two or three levels down and keep about 29 levels. |
| 3 | String length: UTF-8 bytes of the decoded value | **65,536** (64 KiB) | Large enough for any human-readable text a document should embed (descriptions, rationale, error messages). Anything larger is content and must be referenced by digest. At 1/16 of the size ceiling, a document cannot be one giant string. |
| 4 | Member-name length, in bytes (member names are ASCII, D1.2) | **2,048** | Extension identifiers are used as member names in the extensions element (D3.1), so member names must hold an identifier (row 5). |
| 5 | Identifier length: type, extension and schema identifiers, and identity-reference components (D8.2), in bytes | **2,048** | SPIFFE sets 2048 bytes as its interoperability bound for URI identities. It is also a common practical URI limit. Core identifiers under D6.4 are far shorter. |
| 6 | Array length: elements per array | **4,096** | Bounds quadratic uniqueness checks to about 8.4 million comparisons in the worst case. Larger collections should be split across documents linked by digest. |
| 7 | Members per object | **1,024** | Bounds duplicate detection and JCS sorting, even under worst-case hash collisions. |
| 8 | Total JSON values in the document | **100,000** | Realistic metadata averages well over 10 bytes per value, so this binds only pathological documents made of tiny values. A 1 MiB document of bare `0,` values would otherwise hold more than 500,000 values. |
| 9 | Extensions per document, and critical-list entries | **64** each | Each understood extension invokes a handler and possibly a schema. Real documents are expected to carry few. The critical list can never be longer than the extensions element, because its entries must be unique and present (D3.4). |
| 10 | Entries per digest set | **8** | One mandatory `sha256` plus room for algorithm agility, bounding hash work per reference (D4.4). |
| 11 | Number-token length, in bytes | **17** (implied by D1.2) | Not a new limit. Stated so consumers can reject long tokens lexically before any conversion. |

**Validation pipeline.** The proposed order of checks (D7.4):

1. **Stage 0, transport.** Read at most ceiling + 1 bytes. If more are
   available, reject without parsing.
2. **Stage 1, single streaming pass.** Check the BOM, UTF-8 validity, the
   D1.2 number rules, duplicate member names, and every ceiling in rows 2 to
   11. Memory is bounded by depth plus the duplicate-name set of the open
   objects, all of which are capped.
3. **Stage 2, protocol limits.** Read the type identifier and protocol
   version. Reject an unknown type, an unknown major version or an
   unimplemented version (D2). Apply the protocol's own limits, which are
   at most the ceiling. This can use a second pass or the tree from stage
   1, because the ceiling already bounds its cost.
4. **Stage 3, schema validation** against the declared version's
   normative schema (D1.3).
5. **Stage 4, semantic checks**, critical extensions, then canonicalization,
   hashing and reference resolution.

A non-streaming implementation conforms if it enforces stage 0 before
parsing, configures its parser so that it cannot exhaust the stack within
the ceiling, and completes stage 1 checks before stage 3. The size ceiling
bounds its memory.

**Limits and versions.** A protocol's limits are part of its major version.
They change only with a new major version. Under D2.4, a minor version may
add only optional members and values of open members. Raising or lowering a
limit is neither, so allowing it in a minor version would conflict with
ADR 0002. Raising the shared ceiling requires a new ADR.

**Local stricter refusal.** A deployment may refuse documents that are
within the limits, for example on a constrained device. It must record
such a refusal as a local resource refusal, distinct from invalidity. It
must not partially process the document. It must not present the document
as invalid to other parties. Refusal fails closed and so cannot weaken a
decision. Its only cost is availability.

### Unresolved dependencies

- **Per-protocol values.** Each protocol specification states its own
  limits. Tighter values can only be justified once fields exist.
- **Artifact size and reference-resolution budgets.** These are outside
  document limits. Opaque artifacts can be large. Consumers need a
  resolution budget, and a reference left unresolved because the budget ran
  out is unverifiable, never verified (C5, D2.7). The values belong to the
  Evidence Receipt specification and OQ-19.

### Proposed conformance cases

| # | Case | Expected result |
|---|---|---|
| L-01 | Document of exactly 1,048,576 bytes, otherwise valid | Passes the ceiling. Protocol limits apply next. |
| L-02 | Document of 1,048,577 bytes | Reject at stage 0, without parsing. |
| L-03 | Nesting depth exactly 32 (top-level object counts as 1) | Passes the ceiling. |
| L-04 | Nesting depth 33, inside extension data | Reject. Extension data counts toward depth. |
| L-05 | String whose decoded value is 21,845 three-byte UTF-8 characters plus one ASCII character (65,536 bytes), with some characters written as `\u` escapes in the source text | Passes. The measure is decoded UTF-8 bytes, not source bytes, code points or UTF-16 units. |
| L-06 | String of 21,846 three-byte UTF-8 characters (65,538 bytes) | Reject. In UTF-16 it is only 21,846 code units; a consumer measuring UTF-16 would wrongly accept it. |
| L-07 | Array of 4,097 elements | Reject. |
| L-08 | Object of 1,025 members with distinct names | Reject. |
| L-09 | 100,001 values in total, each array and object within its own limits | Reject. |
| L-10 | Number token of 18 characters, for example `-00000000000000001` | Already invalid under D1.2 (leading zeros). Rejected lexically, before any conversion. |
| L-11 | Digest set with 9 entries | Reject. |
| L-12 | Document within the ceiling but above its protocol's tighter limit | Reject at stage 2. |
| L-13 | Consumer refuses a document within all limits under a local policy | Recorded as a local resource refusal, not as invalid. No partial processing. |

### Residual risks

- **Values are estimates.** Ceilings reflect expected document shapes
  before any fields exist. If a protocol needs more, it needs a new ADR to
  raise the ceiling. Values that prove too generous can be tightened per
  protocol at its next major version.
- **Distributed amplification.** One document within the limits can still
  reference many artifacts. Resolution budgets mitigate this (see above).
- **Implementation drift.** Streaming checks are easy to get subtly wrong,
  for example counting depth from 0 instead of 1. Shared test vectors are
  needed once fields exist, as ADR 0002 already notes for JCS.

---

## OQ-5: Identity model

> ADR 0001: "Which kinds of principals exist (human, service, agent,
> runtime, host)? How are they identified? How does the runtime integrate
> with external identity providers?"

### Requirements

- **R5.1 Name every actor.** Every protocol names actors: the principal of
  a task, the proposer of an action, the holder of a capability, the
  producer of a receipt ([dependency map](dependency-map.md)).
- **R5.2 Identification is not authority** (I1, I3, I8). No entity gains
  permission by being identified, by identifying itself, or by being
  described.
- **R5.3 Separate roles.** Identity, authentication, authorization,
  delegation, capability holding and signing are different things. A
  document must never let one stand in for another (by analogy with C7).
- **R5.4 Model- and host-neutral** (C14). The core identity model assumes no
  provider, harness or identity system.
- **R5.5 External identity systems through adapters** (D5.3, D5.5).
  Metadata from OIDC, SPIFFE, MCP or A2A is never a CONTROL decision by
  itself.
- **R5.6 Exact, stable comparison.** Identity references are compared
  exactly (D1.2). An identifier is never reassigned to a different entity.
- **R5.7 Privacy.** Documents are immutable while retained (D2.7), so an
  identity reference may live a long time. Personal data in identifiers
  conflicts with redaction (OQ-19).
- **R5.8 Ready for signing.** When signing is defined (OQ-4 residual), keys
  must bind to identities without changing what an identity is.

### Distinctions

The proposed decision (D8.1) separates the following concepts. One entity
may hold several roles. Holding one role never implies another.

| Concept | Meaning | What it is not |
|---|---|---|
| **Entity** | Anything that can act or be acted for: a person, a system, an agent. | Not a role. |
| **Identifier / identity reference** | A name for an entity: an `(issuer, subject)` pair (D8.2). | Not proof that anyone is that entity. |
| **Credential** | Something an entity presents to an issuer or verifier to prove an identity: an OIDC ID token, a SPIFFE SVID, a client certificate. External to the protocols. | Never carried in protocol documents (C12). |
| **Authenticated principal** | An identity that CONTROL itself has authenticated, by a stated method, at a stated boundary, for a stated interaction. | Not permanent. A recorded authentication is a fact about one interaction, not a reusable credential. |
| **Principal** | ADR 0001: "a human or system identity on whose behalf, or with whose authority, a task runs". Authority traces back to principals and policy. | A principal's authority is still evaluated by CONTROL. It is never implied by identity alone. |
| **Actor** | The entity that directly performs a step, for example the entity that submits a proposal. | Not necessarily the principal. |
| **Agent** | An entity whose proposals are produced by a model through an agent harness. | Never a source of authority (D8.7). |
| **Model** | The AI model inside an agent. | Not an entity kind (D8.4). Described by attributed, non-authoritative descriptors. |
| **Delegate** | An actor acting on behalf of a principal, under a delegation that CONTROL recognizes. | Delegation never expands authority (I8). Its scoping and attenuation are OQ-7. |
| **Capability holder** | The entity a Capability Manifest names as holding capabilities. | Holding a declaration is not authorization (C7). Possessing the manifest document confers nothing. |
| **Approver** | A principal external to the requesting actor who approves or rejects (I8, I9). | Cannot be an agent (D8.7). |
| **Producer** | The party that emitted a document. A migrated or transformed document is attributed to the party that migrated or transformed it (D2.7, D4.5). | Not the signer, and not the authorizer, unless separately established. |
| **Signer** (reserved) | A key holder that endorses content, once signing exists (OQ-4 residual). | Shows endorsement only. Never authorization or correctness (D4.1). |
| **Verifier** | Evaluates claims against evidence. Its trustworthiness is OQ-18. | Not decided here. |
| **Authorization** | A CONTROL decision that a specific proposed action is permitted (C7). | Not an identity property. Identity is one of its inputs. |

### Alternatives

**Identifier form**

| Option | Description | Assessment |
|---|---|---|
| **I1. Issuer-scoped pair `(issuer, subject)`** | The issuer is a URI naming the identity system. The subject is an opaque string, unique and never reassigned within that issuer. | Matches OIDC, which requires `iss` plus `sub` as the account key, with `sub` "locally unique and never reassigned within the Issuer" **(corroborated)**. Maps directly onto SPIFFE trust domains. Makes it explicit which issuer vouches for which subject. **Recommended.** |
| **I2. Single global URI per entity** | For example a SPIFFE ID, a `mailto:` URI or a DID. | Mixes identity systems in one string space and hides which issuer is relied on. Email addresses can be reassigned. Expressible within I1 by using the identifier as subject under a stated issuer. |
| **I3. Runtime-local opaque identifiers** | Each runtime keeps its own mapping table. | Not portable across runtimes or hosts (Task Capsule portability, OQ-16). |
| **I4. Public key as identity** | An entity is identified by a key fingerprint. | Conflates signer and identity (R5.3, R5.8). Rotation changes the identity. Rejected. |
| **I5. Decentralized identifiers or verifiable credentials** | DIDs and VCs. | Out of scope per the standards matrix. Could be used as an issuer under I1 through an adapter later. |

**Entity kinds**

| Option | Kinds | Assessment |
|---|---|---|
| **K1. Fine-grained kinds** | human, service, agent, runtime, host, model, organization | Mixes what an entity *is* with the role it plays. Runtime and host are roles that service entities play. A model is not an actor (I1). Organizations act only through humans or services. |
| **K2. Three kinds, separate roles** | human, service, agent | Each kind has distinct rules (D8.7 restricts agents). Roles are separate. **Recommended.** |
| **K3. No kinds** | — | Loses the ability to state "an agent can never be an approver" as a structural rule. Rejected. |

### Security implications

- **Self-asserted identity.** A model or document can claim any identity.
  If a self-asserted identity could feed an authorization decision, a
  prompt-injected model could claim to be a privileged principal. Only
  identities that CONTROL authenticated itself, or that come from an issuer
  CONTROL is configured to trust, may be decision inputs (D8.5).
- **Kind spoofing.** If kind were self-declared, an agent could declare
  itself `human` and act as an approver. Kind must come from CONTROL's
  registration of the issuer or entity, never from the actor's own content
  (D8.4).
- **Confused deputy and impersonation.** An agent using its principal's
  identity, for example by acting with the principal's token, erases the
  distinction between what the principal did and what the agent did. Every
  proposal is attributed to an actor and, separately, to the principal on
  whose behalf it acts (D8.8).
- **Unverified delegation chains.** RFC 8693 treats prior actors in nested
  `act` claims as informational only and excludes them from access control
  **(corroborated)**. Recent proposals point out that this leaves the path
  of authority unverifiable. A chain entry contributes to authorization only
  if CONTROL validated that hop's delegation (D8.8). Otherwise it is
  recorded as informational.
- **Cross-issuer collision.** Subject `alice` at issuer A is not subject
  `alice` at issuer B. Pair comparison prevents accidental merging. SPIFFE
  makes the same point: assertions must be "qualified by the trust domain"
  (SPIFFE-ID §4.1.2, verified).
- **Identifier reassignment.** If an issuer reassigns a subject, historical
  records then name the wrong entity. Issuers relied on for principals must
  guarantee non-reassignment. OIDC requires this of `sub`. Email addresses
  do not guarantee it.
- **Recorded identity is not authentication.** An identity reference inside
  a receipt or decision record is the producer's statement. Treating it as
  authenticating anyone would turn records into credentials (by analogy
  with C17).
- **Model descriptors.** A model's provider, name and version are reported
  by the harness or host. They are attested at best (I4, I5). A restriction
  based on them ("only model X may propose this") is security-relevant
  (D2.5) but gives no assurance unless the descriptor is authenticated. A
  permission based on them would let any harness that misreports the model
  gain authority, which is forbidden (D8.7).
- **Privacy.** Immutable records containing names or email addresses are
  hard to redact. Opaque subjects keep personal attributes out of core
  identity references (D8.12).

### Recommendation

1. Adopt the distinctions above as normative vocabulary (D8.1).
2. Use **issuer-scoped identity references** `(issuer, subject)` (I1),
   compared exactly as a pair, with both components ASCII and each at most
   2,048 bytes (D7, row 5) (D8.2).
3. Establish trust in an issuer only through CONTROL's local configuration,
   never through document content (D8.3).
4. Adopt **three entity kinds**: `human`, `service` and `agent` (K2). Kind
   comes from CONTROL's registration, never from self-assertion. Models
   and organizations are not kinds. Runtime, host, verifier and approver
   are roles (D8.4).
5. Record **identity provenance** as authenticated, attested or
   self-asserted, mirroring C10. Self-asserted identity is never an input
   that permits (D8.5).
6. **Identification never grants authority.** Authorization needs a CONTROL
   decision over the exact proposal, with a covering capability (I7) and,
   where required, policy or approval (I9). Identity selects which
   capabilities and policies apply. It is not itself a grant (D8.6).
7. **Agents and models are never authority** (D8.7). An agent cannot be an
   approver, a capability grantor or a policy author. Model descriptors may
   only restrict, never permit, and a restriction based on an
   unauthenticated descriptor is recorded as giving no enforcement
   assurance. Whether an agent may be a verifier is left to OQ-18.
8. **Actor and principal are always distinct references.** An actor never
   uses its principal's identity as its own. Delegation hops count only if
   CONTROL validated them. Delegation never expands authority (I8). Its
   mechanics are OQ-7 (D8.8).
9. A Capability Manifest names its holder by identity reference. CONTROL
   considers a capability only for a proposal whose authenticated actor
   matches the holder, or under delegation rules that CONTROL validates
   (OQ-7) (D8.9).
10. Signing, when defined, binds **keys to identity references**. An
    identity is never defined by a key (D8.10).
11. A recorded identity reference is a statement by the record's producer,
    never an authentication (D8.11).
12. Subjects SHOULD be opaque and pseudonymous (D8.12).
13. External identity systems connect through adapters (D5.3) (D8.13):
    - **OIDC:** `iss` maps to issuer and `sub` maps to subject.
    - **SPIFFE:** issuer is `spiffe://{trust-domain}`, and subject is the
      full SPIFFE ID.
    - An adapter's mapping counts as authenticated only if CONTROL
      validated the underlying credential itself. Otherwise it is
      attested.

### Unresolved dependencies

- **OQ-4 residual:** key binding, rotation and revocation for signers.
- **OQ-7:** delegation scope, attenuation, expiry and revocation.
- **OQ-6:** how policy refers to identities and attributes.
- **OQ-9:** approver eligibility beyond the agent prohibition, and quorum.
- **OQ-18:** whether, and under what constraints, an agent can be a
  verifier.
- **OQ-19:** retention and redaction of identity references.
- **OQ-21:** recording the boundary and method of authentication alongside
  the integration grade.

### Proposed conformance cases

| # | Case | Expected result |
|---|---|---|
| ID-01 | A proposal's content claims its actor is a human principal | Self-asserted. Never used to permit. The actor is the authenticated identity of the submitter. |
| ID-02 | Same subject string under two different issuers | Two different identities. |
| ID-03 | An entity registered as `agent` is recorded as an approver | CONTROL rejects the approval. The approval cannot satisfy I8 or I9. |
| ID-04 | An agent submits a proposal using its principal's identity as actor | Rejected as impersonation. The actor must be the agent's own identity. |
| ID-05 | A delegation chain with an intermediate hop that CONTROL did not validate | The hop is recorded as informational. It contributes no authority. |
| ID-06 | A Capability Manifest naming holder H is presented by actor A ≠ H | The capability is not considered for A, unless CONTROL validates a delegation (OQ-7). |
| ID-07 | Policy permits an action because a harness reports model X | Invalid policy. Model descriptors may not permit. |
| ID-08 | An Evidence Receipt names a principal, and a consumer uses that as proof of who the principal is | Non-conformant. A recorded identity is not an authentication. |

### Residual risks

- **Issuer compromise.** If a trusted issuer is compromised, every identity
  it vouches for is compromised. This is unavoidable in any federated
  model. It is mitigated by narrow issuer trust configuration (D8.3).
- **Attested identity at the Observer grade.** The runtime may only ever
  see host-attested identities. Decisions remain records, not enforcement
  (C16), and the records must say the identity was attested.
- **Three kinds may prove too few.** A new kind requires a new ADR, because
  kind affects decisions (D2.5).

---

## OQ-29: CONTROL decision records

> ADR 0002: "How are these decision records identified and carried, and
> how do later records reference them?" It must not be resolved by adding a
> fifth core protocol without a new ADR amending ADR 0001.

This section builds on
[foundations-analysis.md § OQ-29](foundations-analysis.md#oq-29-open-control-decision-records).
That section states requirements R29.1 to R29.6 and options A to F. The
owner asked for a concrete resolution: independent, auditable decision
records that can be referenced by digest without becoming a fifth core
protocol.

### Boundaries that every option must keep

These boundaries come from the owner's instructions and from ADR 0001 and
ADR 0002 (C6, C7, C16, C17):

- **B1.** Action IR is a proposal, not authority.
- **B2.** An Evidence Receipt is not an authorization credential.
- **B3.** A decision record does not grant permission merely by being
  presented.
- **B4.** Authorization is evaluated by CONTROL for the exact proposed
  action, and enforced only at an actual controlled execution boundary.
- **B5.** No new core protocol, and no change to ADR 0001 or ADR 0002.

### The central observation

Enforcement never needs to *read* a decision record. Under B4, the
enforcement point obtains authorization from CONTROL's own evaluation of
the exact proposal, inside the controlled boundary. The decision record is
an **audit artifact**: a durable, digest-addressable statement that CONTROL
made a decision. It is written when the decision is made and read later, by
TRUST and by auditors.

That changes the analysis of the alternatives:

- The **timing objection** to option A disappears. The record does not
  need to exist before execution to authorize it, because nothing is
  authorized by records. It still should be produced at decision time,
  because that is when CONTROL knows the decision's inputs.
- ADR 0001 places **Audit** in the TRUST plane, as "a durable record of
  decisions, actions and outcomes". A record *of* a CONTROL decision
  therefore belongs, architecturally, with TRUST › Audit. What must stay in
  CONTROL is the decision itself, and the authority to make it.

### Alternatives

| Option | Description | Meets B1 to B5? | Assessment |
|---|---|---|---|
| **A. Inside Evidence Receipts only** | No separate record. The decision appears only inside the receipt about the execution. | B5 yes. Fails R29.1. | Denials of actions that never execute would have no natural receipt. A decision made before execution would have no record until afterwards. Rejected. |
| **B-int. Internal Audit format** | Independent records in an unspecified, runtime-internal format. Evidence Receipts reference them by digest. | Yes. | Not auditable by third parties. A receipt would reference a digest whose content no external verifier can interpret, which makes R29.3 hollow. If the format is specified to fix this, it becomes a protocol in all but name. |
| **B-new. Independent specified format, separately versioned** | A specified decision-record format with its own namespace and version line. | Fails B5. | A fifth core protocol in all but name. Excluded. |
| **C. Execution-bound Action IR form** | Action IR gains an "action as authorized" form. | Weakens B1. | An Action IR document carrying a decision comes close to a credential (C17). Rejected. |
| **D. Inside the Capability Manifest** | Per-action grants recorded in the manifest. | Fails C7. | Rejected. |
| **E. Fifth core protocol** | — | Fails B5. | Excluded. |
| **F. Host logs only** | — | Fails R29.1 for Managed-grade decisions. | Usable only as host-attested evidence. |
| **G. A distinct decision-record form of the Evidence Receipt protocol** | Independent, digest-addressed records produced by CONTROL at decision time. Their format is specified in the Evidence Receipt specification, shares its version line, extension model and limits, and is referenced by digest from later receipts. | Yes. See below. | **Recommended.** |

### Why option G

- **Independent (R29.1).** Each decision is its own document, with its own
  content digest (D4.3). It is produced when the decision is made, whether
  or not anything executes. Denials, rejections and permits are all
  recorded (R29.6).
- **Bound to the exact proposal (R29.2).** It references the proposal by
  content digest, and every input it relied on by digest or identity
  reference: Capability Manifests, policy (OQ-6), approvals (OQ-9), risk
  classification (OQ-8), and the authenticated actor and principal (D8).
- **Auditable (R29.3).** Its format is specified, versioned and governed by
  ADR 0002's rules, so third-party verifiers can interpret it. Evidence
  Receipts reference it by digest.
- **Not a fifth protocol (B5).** It is specified *within* the Evidence
  Receipt protocol. It has no namespace root of its own, no separate version
  line (D2.1) and no new conformance class. ADR 0001 states the Evidence
  Receipt's purpose as a "machine-verifiable record linking actions,
  evidence, claims and verification". A decision record links an action
  (the proposal digest) to the CONTROL evaluation, which is evidence of the
  CONTROL step. It fits within that frozen purpose, inside the TRUST ›
  Audit component that ADR 0001 already defines.
- **Not a credential (B2, B3, R29.4).** The same rule as C17 applies, and is
  stated as a binding requirement (D9.4).
- **Grade-aware (R29.5).** A decision record never claims enforcement.
  Enforcement facts appear only in later Evidence Receipts, together with
  the integration grade and boundary (C10, C16).
- **Action IR stays a pure proposal (B1).** No execution-bound Action IR
  form is needed. Proposal, decision and evidence are linked by Evidence
  Receipts and decision records, which are produced afterwards. This removes
  OQ-29's conditional block on Action IR.

**Objection: "this mixes a CONTROL outcome into a TRUST record."** This was
the objection to option A in the ADR 0002 analysis. Under G, CONTROL still
makes the decision and holds its authority. The TRUST-plane document only
*records* that the decision was made. ADR 0001 itself puts records of
decisions in TRUST › Audit. What would violate the planes is TRUST making,
changing or conveying an authorization. G forbids all three.

**Governance test: what keeps G from becoming a protocol in all but name.**
The proposed decision makes these conditions binding. If any of them must
be broken, a new ADR amending ADR 0001 is required (D9.9):

1. Decision records are specified only in the Evidence Receipt
   specification.
2. They share the Evidence Receipt protocol's version (D2.1), extension
   element and limits. No independent version line.
3. They have no namespace root of their own. Their identifier, if any, sits
   under `{root}/evidence-receipt/` (D6.4).
4. Their only producer role is the CONTROL plane of a runtime. Models,
   agents and hosts never produce them.
5. They create no new conformance class separate from Evidence Receipt
   conformance.

### Security implications

- **Credential misuse.** This is the main risk. Mitigation: an enforcement
  point accepts authorization only from CONTROL's own evaluation of the
  exact proposal, received over a channel inside the controlled boundary
  and never relayed through the requesting actor, model or host. It never
  accepts a decision record, a receipt, or a copy of either, from anyone
  (D9.4).
- **Proposal substitution.** Between decision and execution, an attacker
  could swap the action. The enforcement point must confirm that the action
  it executes has the same content digest as the proposal CONTROL evaluated
  (D9.5).
- **Replay.** A permit for proposal P does not authorize a second execution
  of an identical P. Each execution attempt needs its own CONTROL
  authorization. An earlier permit record is never sufficient (D9.5).
  Expiry and single-use semantics belong to OQ-9.
- **Forgery without signatures.** There is no signing yet (D4.6). A
  decision record is trustworthy only inside the trust boundary that
  produced it. Elsewhere it is a statement attributed to its producer,
  without authenticity (D9.8). Because records are never credentials,
  forging one cannot authorize anything. It can only mislead an audit,
  which is detectable when the auditor holds digests from an independent
  source (D3.7).
- **Missing or deleted records.** A receipt whose decision reference cannot
  be resolved cannot show that the action was authorized. That claim is
  unverifiable, never verified (C5, D2.7).
- **Overclaiming enforcement.** At the Observer grade, decisions are still
  recorded, but nothing is prevented. A denial record must not claim
  prevention (C16).
- **Secrets and privacy.** Inputs are referenced by digest, never embedded
  (C12). Identity references follow D8.12.

### Recommendation

Adopt **option G** as the resolution of OQ-29, with the normative rules
D9.1 to D9.9 in ADR 0003.

Effect on the dependency map, **if accepted**:

- Action IR is no longer conditionally blocked by OQ-29, because no
  execution-bound Action IR form is introduced.
- The Evidence Receipt specification must define the decision-record form.
  It depends on OQ-6, OQ-8 and OQ-9 for how decision inputs are
  identified.
- The Capability Manifest and Task Capsule are unaffected.

### Unresolved dependencies

- **OQ-6:** how a policy and its version are identified in a decision.
- **OQ-8:** how a risk classification is recorded.
- **OQ-9:** approval binding, expiry, single use and quorum. Approval events
  may be their own decision records.
- **OQ-17:** the outcome vocabulary. D9 requires at least distinct permit and
  deny outcomes, and fixes no complete set.
- **OQ-20:** audit integrity, for example hash chaining of decision records.
- **OQ-21:** how the grade and boundary in effect are recorded.
- **OQ-4 residual:** authenticity across trust boundaries.

### Proposed conformance cases

| # | Case | Expected result |
|---|---|---|
| DR-01 | An actor presents a genuine permit decision record to an enforcement point | Not accepted as authorization. Enforcement requires CONTROL's own evaluation. |
| DR-02 | An Evidence Receipt containing a permit reference is presented as permission | Not accepted (C17). |
| DR-03 | CONTROL permits proposal P, and the action delivered for execution has a different digest | Not executed. Substitution detected (D9.5). |
| DR-04 | A second execution of an identical P, citing the first permit record | Requires a new CONTROL authorization (D9.5). |
| DR-05 | A denied proposal never executes | A deny decision record exists and is referenceable by digest. |
| DR-06 | A receipt references a decision record that has been deleted | "Authorized" is unverifiable, never verified. |
| DR-07 | A decision record, at the Observer grade, states that the action was prevented | Invalid. Decision records never claim enforcement. |
| DR-08 | A decision record produced by an agent or a host | Invalid. Only CONTROL produces decision records. |
| DR-09 | A decision record whose version differs from the Evidence Receipt protocol version line | Invalid. Decision records share the protocol's version line (D9.9). |

### Residual risks

- **Producer-side misuse.** A deployment could still build an enforcement
  point that reads records as credentials. That would be non-conformant,
  and only conformance testing and review can catch it.
- **Interpretation of ADR 0001.** Whether a decision record fits within the
  Evidence Receipt's frozen purpose is a judgment. The analysis above
  argues that it does. Accepting D9 is the owner's confirmation of that
  reading. If the owner disagrees, the remaining compliant option is
  B-int, which gives up third-party auditability.

---

## Sources and verification

The session that prepared this analysis had restricted network access. The
RFC Editor, the IETF Datatracker, `openid.net` and `w3id.org` were blocked
by its egress proxy. GitHub raw content was reachable. The verification
levels are those of the
[standards matrix](standards-matrix.md#verification-levels). The matrix
itself is not changed by this proposal. If ADR 0003 is accepted, rows for
the newly cited sources should be added there.

| Source | Used for | Verification |
|---|---|---|
| RFC 4151, `tag` URI scheme | OQ-26, option C | **Corroborated** (search excerpt): syntax `tag:` authorityName `,` date `:` specific; the tagging entity must control the domain or email at 00:00 UTC on the date; no authoritative resolution mechanism. |
| RFC 8141, URNs | OQ-26, option D | **Unverified**: formal namespace registration with IANA. |
| RFC 2606 and RFC 6761, reserved `.invalid` | OQ-26, placeholder | **Unverified** in this session. |
| w3id.org README ([perma-id/w3id.org](https://github.com/perma-id/w3id.org)) | OQ-26, option B | **Verified (primary)**: consortium management, HTTPS-only, "intended to be around for as long as the Web is around", changes by pull request, administrators may deny generic names. |
| SPIFFE-ID ([spiffe/spiffe](https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE-ID.md)) | OQ-27, row 5; OQ-5 | **Verified (primary)**: 2048-byte interoperability bound (§2.3); trust domain names are self-registered and may collide (§2.1.1); assertions are qualified by the trust domain (§4.1.2). |
| RFC 8259 §9 | OQ-27 | **Corroborated** (search excerpt): implementations may limit text size, nesting depth, number range and string length. |
| Parser defaults: [`System.Text.Json`](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Text.Json/src/System/Text/Json/Reader/JsonReaderOptions.cs), [`serde_json`](https://github.com/serde-rs/json/blob/master/src/de.rs), [Ruby `json`](https://github.com/ruby/json/blob/master/ext/json/ext/parser/parser.c), [Jackson 2.18](https://github.com/FasterXML/jackson-core/blob/2.18/src/main/java/com/fasterxml/jackson/core/StreamReadConstraints.java), [Go `encoding/json`](https://github.com/golang/go/blob/master/src/encoding/json/scanner.go), [CPython](https://github.com/python/cpython/blob/main/Include/internal/pycore_ceval.h) | OQ-27 | **Verified (primary)**: default constants read from source on the default branches as of 2026-10-02. That CPython's `json` decoder is bounded by the recursion limit is **unverified** in this session. Defaults may change in later releases. |
| OpenID Connect Core 1.0, `sub` claim | OQ-5 | **Corroborated** (search excerpt): `sub` is locally unique and never reassigned within the issuer, at most 255 ASCII characters; `iss` plus `sub` is the account key. |
| RFC 8693, `act` claim | OQ-5 | **Corroborated** (search excerpts and IETF drafts citing it): prior actors in nested `act` claims are informational only and not used for access control. |
