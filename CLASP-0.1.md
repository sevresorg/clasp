# CLASP 0.1

**Clause-Level Access and Standards Protocol**

Draft, May 2026. Initial working draft. Substantive change is expected before the protocol is considered stable.

---

## Table of Contents

1. [Purpose](#purpose)
2. [Scope](#scope)
3. [Relationship to CoMP](#relationship-to-comp)
4. [Layering: CoMP handshake versus MCP tool layer](#layering-comp-handshake-versus-mcp-tool-layer)
5. [Terminology](#terminology)
6. [Naming convention used in this document](#naming-convention-used-in-this-document)
7. [Extension: `aisystemuse.ext`](#extension-aisystemuseext)
8. [Extension: `scope.ext`](#extension-scopeext)
9. [Extension: `retrieval.ext`](#extension-retrievalext)
10. [CLASP tool semantics](#clasp-tool-semantics)
11. [Citation Envelope](#citation-envelope)
12. [Discoverability Catalog](#discoverability-catalog)
13. [Cross-publisher reference resolution](#cross-publisher-reference-resolution)
14. [Session revocation and envelope durability](#session-revocation-and-envelope-durability)
15. [Multi-language behavior](#multi-language-behavior)
16. [Deliverable type and session strictness](#deliverable-type-and-session-strictness)
17. [Examples](#examples)
18. [Open Issues](#open-issues)

---

## Purpose

CLASP extends the IAB Tech Lab Content Monetization Protocols (CoMP) v1.0 specification to address the specific needs of engineering and technical standards. CoMP's base handshake is preserved without modification; CLASP populates the `ext` placeholders that CoMP defines on every object and additionally specifies a free discoverability tier that sits outside CoMP.

The protocol is intended to allow:

- Engineering standards bodies (the Content Owners) to expose their corpus to licensed AI agents in a structured, addressable, and machine-readable way.
- AI agents (the AI Systems) to retrieve specific clauses of specific editions of specific standards, with the citation metadata necessary for downstream attribution in human-facing deliverables.
- Practicing engineers, working through agentic tools, to consult authoritative standards without violating licensing terms.
- AI agents and operators to discover what standards exist, at the level of metadata, without licensing them first.

CLASP does not redefine commercial models, replace existing subscription mechanisms, or impose architectural choices on standards bodies beyond what is necessary for interoperability.

## Scope

### In scope

- Vocabulary for engineering-specific intended uses, populated into `aisystemuse.ext`.
- Vocabulary for standards-specific content metadata, populated into `scope.ext`.
- Capability declaration for CLASP-conformant MCP servers, populated into `retrieval.ext`.
- The normative specification of the free discoverability catalog that sits adjacent to CoMP.
- A citation envelope that CLASP-conformant retrieval responses MUST attach to returned content.
- Tool-level semantics sufficient to define the response shape and citation discipline of each CLASP tool.
- Session revocation behavior and envelope durability rules.
- Multi-language retrieval and citation semantics.

### Out of scope

- Bot blocking strategy, deferred to the Content Owner's CDN or edge infrastructure (consistent with CoMP).
- Commercial terms and pricing models, negotiated bilaterally between Content Owner and AI System.
- Token issuance, metering, and clearing, which remain implementation choices.
- The full MCP tool wire schemas, specified separately in Norma.
- Aggregator catalogs that walk multiple publisher catalogs (see [Open Issues](#open-issues)).

## Relationship to CoMP

CLASP is strictly additive to CoMP. Every CLASP exchange is a valid CoMP exchange. CoMP-only AI Systems that do not understand CLASP extensions will receive a valid Package response and can fall back to the base retrieval interface declared in `retrieval.type` (typically MCP, value 3, per CoMP's Endpoint / Delivery Format list).

The discoverability catalog is the one CLASP capability that lives outside CoMP. CoMP intentionally leaves supply discovery to direct AI-System-to-Content-Owner contact; CLASP fills that gap with a lightweight, unauthenticated, HTTP-cacheable catalog.

## Layering: CoMP handshake versus MCP tool layer

CLASP separates two layers of interaction, plus an unauthenticated discovery tier that precedes both:

**Layer 0: Discoverability catalog.** AI Systems consult the catalog at `https://<publisher-domain>/.well-known/clasp-catalog` to learn what standards a publisher offers, what editions are current, what scope each covers, and where to license. The catalog is unauthenticated and MUST NOT carry normative text.

**Layer 1: CoMP handshake.** The AI System sends an `aisystem`/`aisystemuse` object; the Content Owner returns a `package` object containing a `retrieval` object. This layer authorizes a session and communicates metadata about the licensed scope, the standard's identity, available languages, and the available tools.

**Layer 2: MCP tool layer.** Inside the authorized session, the AI System makes MCP tool calls against the endpoint declared in `retrieval.endpoint`. Each tool call carries its own arguments. Each tool response is accompanied by a Citation Envelope.

The fields specified in `aisystemuse.ext` parameterize the session as a whole. They are not per-tool-call arguments.

## Terminology

- **Standard**: A document published by a Standards Developing Organization (SDO) that specifies normative requirements for engineering practice.
- **Designation**: The canonical identifier of a standard. Example: `PASC-188.01`.
- **Edition**: A versioned release of a standard. An edition is one logical thing; multi-language renderings are renderings of the same edition.
- **Clause**: An addressable section of a standard. Identifiers are hierarchical (e.g., `5.2.3`) and may include annex references.
- **Canonical language**: The language an edition's publisher considers authoritative for normative interpretation.
- **Rendering**: A specific language version of an edition's text.
- **Normative reference**: An external document that a clause requires for full conformance.
- **Defined term**: A term whose meaning is established within the body of a standard.
- **Citation envelope**: The metadata object returned with every retrieval response, establishing provenance and supporting refusal to emit unattributed normative content downstream.
- **Session**: A CoMP-handshake-authorized interaction between an AI System and a Content Owner.
- **Revocation**: A Content Owner action that ends a license before its natural expiration.
- **Catalog**: The unauthenticated discoverability tier at `https://<publisher-domain>/.well-known/clasp-catalog`.

## Naming convention used in this document

The standards designations, publisher domains, and SDO names used in this document and in the accompanying examples are fictional.

| Fictional name | Acronym | Domain | Niche |
| --- | --- | --- | --- |
| Process Automation Standards Council | PASC | `pasc.example` | Process control, batch and continuous control models, safety instrumented systems |
| Technical Methods Federation | TMF | `tmf.example` | Test methods, verification guides, qualification practices |
| Bioprocess and Pharmaceutical Equipment Council | BPEC | `bpec.example` | Bioprocess and pharmaceutical equipment standards |
| International Electronics Standards Federation | IESF | `iesf.example` | Electrical and electronics, control programming languages, fieldbus |

The `.example` TLD is reserved by IANA for documentation.

## Extension: `aisystemuse.ext`

Engineering-standards intended-use vocabulary, populated into the `ext` field of a CoMP `AISystemUse` object. All fields are optional unless noted.

| Attribute | Type | Description |
| --- | --- | --- |
| `clasp_version` | string, required | CLASP profile version. Use `"0.1"` for this draft. |
| `engineering_intent` | int, array | Engineering-specific intent values; see [List: Engineering Intent](#list-engineering-intent). Used in addition to (not in place of) CoMP `function`. |
| `target_designation` | string, array | Specific standard designation(s) the AI System is requesting. |
| `target_edition` | string | Specific edition of the target designation. For comparison tools, interpreted as the "from" edition. |
| `target_edition_compare` | string | Additional edition for comparison tools (`list_changes`). |
| `target_clauses` | string, array | Specific clauses requested; see [Clause identifier syntax](#clause-identifier-syntax). |
| `target_clauses_numbering` | string | When both `target_edition` and `target_edition_compare` are populated, identifies which edition's numbering `target_clauses` references. Values: `"from"` or `"to"`. Default: `"to"`. MUST NOT be populated when `target_edition_compare` is empty. |
| `preferred_languages` | string, array | Ordered list of BCP 47 language tags the AI System prefers for retrieved renderings. |
| `requester_sectors` | int, array | Industry sector(s) of the requester. Uses the same values as [List: Industry Sector](#list-industry-sector). Allows Content Owners to apply sector-conditional rules. Multi-sector requesters may declare multiple values. |
| `deliverable_type` | int | The kind of human-facing artifact the AI System is helping produce; see [List: Deliverable Type](#list-deliverable-type). |
| `requester_authority` | int | Identifies whether the license is held by an individual practitioner or an organization; see [List: Requester Authority](#list-requester-authority). |

### List: Engineering Intent

| Value | Label | Description |
| --- | --- | --- |
| 0 | design-consultation | Reading clauses to inform a design decision. |
| 1 | gap-analysis | Comparing an artifact against a standard. |
| 2 | derivative-spec | Producing a customer- or company-specific specification that references the standard. |
| 3 | tooling-integration | Building software tooling that operates against the standard. |
| 4 | training-material | Producing educational or onboarding content about the standard. |
| 5 | regulatory-submission | Supporting a submission to a regulator that cites the standard. |
| 6 | other | Other engineering use. |

### List: Deliverable Type

| Value | Label | Description |
| --- | --- | --- |
| 0 | exploratory | No persistent deliverable; conversational use only. |
| 1 | internal-document | Internal company document not for external distribution. |
| 2 | client-document | Document delivered to an external client. |
| 3 | stamped-deliverable | Engineering document bearing a Professional Engineer seal or equivalent regulatory stamp. |
| 4 | submitted-to-regulator | Document submitted to a government regulator. |

### List: Requester Authority

| Value | Label | Description |
| --- | --- | --- |
| 0 | individual-licensee | The license is held by an identified individual practitioner. |
| 1 | organization-licensee | The license is held by an organization. |
| 2 | unknown | Authority not declared. |

### Relationship to CoMP `function`

The CoMP `function` field carries the commercial-classification intent. The CLASP `engineering_intent` field carries the engineering-workflow intent. CLASP-conformant AI Systems MUST populate both; Content Owners use the union to authorize the session.

### Sector declarations: publisher side and requester side

CLASP carries sector information on both sides of the handshake:

- `scope.ext.industry_sectors` is set by the Content Owner and declares which sectors the **standard** applies to.
- `aisystemuse.ext.requester_sectors` is set by the AI System and declares which sectors the **requester** operates in.

The two are independent. A multi-sector standard (e.g., a safety standard applicable to oil-and-gas, chemical, and power-generation) may serve a requester operating in only one of those sectors. Content Owners MAY use the requester's declared sector to apply sector-conditional rules in `compliance_check` or to filter `search` results. The protocol does not require Content Owners to use `requester_sectors`; declaring it is informational.

## Extension: `scope.ext`

Standards-specific content metadata, populated into the `ext` field of a CoMP `Scope` object on the Content Owner response.

| Attribute | Type | Description |
| --- | --- | --- |
| `clasp_version` | string, required | CLASP profile version of the response. |
| `publisher` | string, required | Canonical identifier of the publishing SDO. |
| `designation` | string, required | Standard designation. |
| `edition` | string, required | Edition identifier. |
| `pubdate` | string | Publication date in ISO-8601. |
| `canonical_language` | string, required | BCP 47 language tag of the authoritative-language rendering. |
| `available_languages` | string, array | BCP 47 language tags of all renderings the Content Owner can serve. MUST include `canonical_language`. |
| `supersedes` | string, array | Designations and editions of standards this edition supersedes. |
| `superseded_by` | string | Designation and edition of any standard that has superseded this one. |
| `industry_sectors` | int, array | Industry sectors the standard applies to; see [List: Industry Sector](#list-industry-sector). |
| `clause_scheme` | string | URI of the clause-numbering scheme. Normative for parsing `target_clauses` syntax. |
| `normative_refs` | object, array | Structured normative references. |
| `defined_terms_uri` | string | Discoverability endpoint for the defined-terms vocabulary of this edition. |
| `regulatory_incorporations` | object, array | Optional; see [Object: RegulatoryIncorporation](#object-regulatoryincorporation). |

### Object: NormativeRef

| Attribute | Type | Description |
| --- | --- | --- |
| `designation` | string, required | Designation of the referenced standard. |
| `edition` | string | Specific edition referenced. |
| `publisher` | string | Publisher of the referenced standard. |
| `clauses` | string, array | Specific clauses normatively invoked. |

### Object: RegulatoryIncorporation

| Attribute | Type | Description |
| --- | --- | --- |
| `regulator` | string, required | Canonical identifier of the regulator. |
| `regulation_id` | string, required | Regulator-assigned identifier for the regulation. |
| `incorporation_date` | string | ISO-8601 date the incorporation took effect. |
| `pinned_edition` | string, required | The specific edition incorporated. |
| `transition_window_start` | string | ISO-8601 date on which `next_pinned_edition` becomes acceptable. |
| `transition_window_end` | string | ISO-8601 date after which `pinned_edition` is no longer acceptable. |
| `next_pinned_edition` | string | Edition identifier of the edition that will become pinned after the transition window. |

#### Multiple incorporations and jurisdictional conflicts

A standard may be incorporated by reference into regulations from multiple regulators. The `regulatory_incorporations` array supports zero or more entries. Content Owners SHOULD populate all known incorporations they are aware of, regardless of jurisdiction.

When an AI System receives multiple `regulatory_incorporations`, including cases where two regulators pin different editions of the same standard, the protocol does not impose a precedence rule. The AI System SHOULD use the deliverable's target (a submission destined for a specific regulator) and the session's `engineering_intent` to determine which incorporation is relevant. If the deliverable target is ambiguous or the deliverable is intended for multiple regulators, the AI System SHOULD surface all relevant incorporations to the engineer rather than picking silently.

This places jurisdictional reasoning where it belongs (in the AI System's session context and the engineer's situation) rather than in the Content Owner's metadata.

### List: Industry Sector

Each value carries a normative ISIC Rev 4 cross-reference.

| Value | Label | ISIC Rev 4 | Notes |
| --- | --- | --- | --- |
| 0 | bioprocess | C21 | Manufacture of basic pharmaceutical products. |
| 1 | pharmaceutical | C21 | Manufacture of basic pharmaceutical products. |
| 2 | chemical | C20 | Manufacture of chemicals and chemical products. |
| 3 | oil-and-gas | B06 | Extraction of crude petroleum and natural gas. Also C19 for downstream. |
| 4 | power-generation | D35 | Electricity, gas, steam and air conditioning supply. |
| 5 | discrete-manufacturing | C25 | Manufacture of fabricated metal products. Spans C25 through C30 broadly. |
| 6 | building-services | F43 | Specialized construction activities. |
| 7 | water-and-wastewater | E36 | Water collection, treatment and supply. Also E37. |
| 8 | food-and-beverage | C10 | Manufacture of food products. Also C11. |
| 9 | semiconductor | C26 | Manufacture of computer, electronic and optical products. |
| 10 | medical-device | C32.5 | Manufacture of medical and dental instruments. |
| 11 | aerospace | C30.3 | Manufacture of air and spacecraft. |
| 12 | automotive | C29 | Manufacture of motor vehicles. |
| 99 | other | (none) | No canonical mapping. |

### Clause identifier syntax

The Content Owner is the authoritative source for clause syntax via the `clause_scheme` URI. CLASP defines the following conventions:

- Single clause: `"5.3.2"`.
- Annex clause: `"Annex C.4"`.
- Range: `"5.3.2..5.3.5"`. Two dots; endpoints inclusive.
- Multiple: pass as separate array elements in `target_clauses`.

Content Owners MUST reject a clause identifier that does not parse against the declared scheme.

#### Edition numbering for `target_clauses` in cross-edition tools

When the session declares both `target_edition` and `target_edition_compare` (i.e., `list_changes` is the intended tool), the clause identifiers in `target_clauses` reference one or the other edition's numbering. The default is the "to" edition (the value of `target_edition_compare`). The AI System MAY override the default by setting `aisystemuse.ext.target_clauses_numbering` to `"from"`.

This matters when editions renumber clauses. If clause `7.5` in the older edition becomes clause `7.2` in the newer edition, an engineer asking about "clause 7.2" probably means the current-edition identifier (the default), but an engineer auditing migration from the older edition may want `7.2` interpreted as the older identifier. Explicit declaration via `target_clauses_numbering` removes ambiguity.

`target_clauses_numbering` MUST NOT be populated when `target_edition_compare` is empty (i.e., for single-edition tools). The Content Owner MUST reject a handshake that violates this constraint.

## Extension: `retrieval.ext`

| Attribute | Type | Description |
| --- | --- | --- |
| `clasp_version` | string, required | CLASP profile version supported by the endpoint. |
| `tools` | int, array, required | CLASP tool capabilities exposed; see [List: CLASP Tools](#list-clasp-tools). |
| `exact_text_mode` | int | Whether the endpoint supports deterministic exact-text retrieval. REQUIRED for endpoints serving stamped or submitted deliverables. |
| `citation_envelope` | string, required | URI of the citation envelope schema this endpoint emits. For v0.1, this MUST be `https://clasp.example/schemas/citation-envelope-0.1.json` (or the implementer's locally-hosted copy). |
| `max_artifact_bytes` | int | Maximum artifact size accepted by `compliance_check`, in bytes. REQUIRED if `compliance_check` is listed in `tools`. |
| `revocation_uri` | string | Optional. URI at which the Content Owner publishes a revocation list. |

### List: CLASP Tools

| Value | Tool | Description |
| --- | --- | --- |
| 0 | get_clause | Retrieve a specific clause by ID. |
| 1 | search | Full-text or semantic search within the licensed scope. |
| 2 | resolve_term | Resolve a defined term to its definition clause. |
| 3 | list_changes | Return the diff between two editions of the same designation. |
| 4 | resolve_reference | Follow a normative reference to its target. |
| 5 | compliance_check | Submit an artifact against a clause or set of clauses; return structured gap analysis. |
| 99 | other | Vendor-specific extension. |

## CLASP tool semantics

### get_clause

**Inputs:** clause identifier(s); optional `mode` (`exact-text` default or `synthesized`); optional `language`.

**Response:** array of clause objects with identifier and content.

**Citation envelope:** one per response; one citation per returned clause; `mode = exact-text` (0) or `synthesized` (1); envelope's top-level `language` identifies the rendering.

### search

**Inputs:** query string; optional filters; optional `max_results`; optional `language`.

**Response:** array of hits ordered by relevance, each with clause identifier, snippet, and relevance score.

**Citation envelope:** one per response; one citation per hit; `mode = snippet` (2).

### resolve_term

**Inputs:** term string; optional `language`.

**Response:** definition clause(s).

**Citation envelope:** one per response; one citation per definition; `mode = exact-text` (0).

### list_changes

**Inputs:** none beyond session-level; optional clause-filter; optional `language`. "From" edition is `target_edition`; "to" is `target_edition_compare`. `target_clauses` references one edition's numbering per `target_clauses_numbering` (default `"to"`); see [Edition numbering for `target_clauses` in cross-edition tools](#edition-numbering-for-target_clauses-in-cross-edition-tools).

**Response:** array of change records, each with clause (in "to" numbering), change_kind, and structured description.

**Citation envelope:** one per response; envelope's `primary_edition` is "from", `comparison_edition` is "to"; one citation per change; `mode = diff` (3).

### resolve_reference

**Inputs:** normative reference identifier; optional inline-vs-pointer hint; optional `language`.

**Response:** clause text if in-scope; structured pointer if held by another Content Owner.

**Citation envelope:** one per response; one citation; `mode = exact-text` (0), `synthesized` (1), or `pointer` (5). Envelope `language` is `null` for pointer responses.

### compliance_check

**Inputs:** artifact (binary or text), content-type, clause set; size MUST NOT exceed `retrieval.ext.max_artifact_bytes`; optional `language`.

**Response:** array of findings (`finding_id`, `severity`, `referenced_clauses`, `artifact_location`, `explanation`).

**Citation envelope:** one per response; one citation per unique referenced clause; `mode = structured-analysis` (4). The envelope's top-level `language` identifies the language of finding `explanation` fields.

#### Multi-language findings

One `compliance_check` envelope carries one language. A deliverable that requires findings in multiple languages (e.g., a global submission with English, Japanese, and Chinese sections) MUST issue one `compliance_check` call per target language, supplying the same artifact each time with a different `language` argument. Each response carries its own envelope; each finding's `explanation` is in the envelope's declared language. The repeated artifact upload is acceptable for the rare multi-language case; it keeps envelope and hash semantics simple (one language per envelope, one explanation per finding).

The Content Owner MUST NOT retain the artifact beyond the duration of the tool call unless the license terms explicitly permit retention.

## Citation Envelope

Every response from a CLASP-conformant retrieval endpoint MUST be accompanied by exactly one citation envelope.

### Schema

```json
{
  "envelope_version": "0.1",
  "tool": "<one of: get_clause, search, resolve_term, list_changes, resolve_reference, compliance_check>",
  "publisher": "<canonical domain>",
  "designation": "<standard designation>",
  "primary_edition": "<edition>",
  "comparison_edition": "<edition or null>",
  "language": "<BCP 47 tag or null>",
  "retrieval_timestamp": "<ISO-8601>",
  "endpoint_uri": "<URI>",
  "license_id_fingerprint": "sha256:<hex>",
  "deliverable_type": <int>,
  "requester_authority": <int>,
  "citations": [
    {
      "clause": "<clause identifier>",
      "mode": <int>,
      "content_hash": "sha256:<hex>",
      "content_length": <int>,
      "snippet": null | { "offset": <int>, "length": <int>, "of_total_length": <int> },
      "diff": null | { "from_edition": "<edition>", "to_edition": "<edition>", "change_kind": "<one of: added, removed, modified, renumbered>" },
      "pointer": null | { "target_publisher": "<domain>", "target_designation": "<designation>", "target_edition": "<edition>", "target_clause": "<clause identifier>", "discoverability_uri": "<URI>" }
    }
  ]
}
```

### Per-citation `mode` values

| Value | Label | Used by | Per-citation variant field populated |
| --- | --- | --- | --- |
| 0 | exact-text | get_clause, resolve_term, resolve_reference (in-scope) | none |
| 1 | synthesized | get_clause, resolve_reference (in-scope) | none |
| 2 | snippet | search | `snippet` |
| 3 | diff | list_changes | `diff` |
| 4 | structured-analysis | compliance_check | none |
| 5 | pointer | resolve_reference (out-of-scope) | `pointer` |

### Hashing rules

`content_hash` is computed over the response body content associated with this citation:

- `mode = exact-text`, `synthesized`: UTF-8 of the clause text returned.
- `mode = snippet`: UTF-8 of the snippet.
- `mode = diff`: canonical JSON serialization (RFC 8785 JCS) of the diff record.
- `mode = structured-analysis`: canonical JSON of the finding object(s) referencing this clause, sorted by `finding_id`.
- `mode = pointer`: canonical JSON of the `pointer` object.

### AI System obligations

An AI System emitting CLASP-retrieved content in a downstream deliverable MUST:

1. Attach the citation envelope (or a faithful subset including all fields above) to the deliverable.
2. Refuse to emit a verbatim clause quotation unless a matching `mode = 0` citation is present.
3. Refuse to emit a clause-by-clause gap analysis unless a matching `mode = 4` citation is present.
4. Refuse to claim that a clause changed between editions unless a matching `mode = 3` citation is present.
5. Refuse to emit content for a deliverable strictness higher than the envelope's recorded `deliverable_type`.
6. When a deliverable has regulatory or stamped weight and the envelope's `language` is not the standard's `canonical_language`, the AI System MUST either re-retrieve the cited clauses in the canonical language and attach a second envelope, or annotate the deliverable to identify the cited text as a translation rendering of the canonical-language clause.

## Discoverability Catalog

CoMP does not specify how an AI System discovers what content is available. CLASP fills that gap with a free, unauthenticated, HTTP-cacheable catalog hosted by each publisher.

### Endpoints

```
https://<publisher-domain>/.well-known/clasp-catalog                  (CatalogIndex)
https://<publisher-domain>/.well-known/clasp-catalog/<designation>    (CatalogStandard)
```

Both MUST be reachable without authentication. Both MUST NOT carry normative clause text, definition text, or any other content the publisher considers normative.

### CatalogIndex shape

```json
{
  "catalog_version": "0.1",
  "publisher": "<canonical domain>",
  "publisher_name": "<human-readable publisher name>",
  "publisher_licensing_overview_uri": "<URI>",
  "standards": [
    {
      "designation": "<designation>",
      "title": "<short title>",
      "current_edition": "<edition>",
      "catalog_uri": "<URI of the CatalogStandard for this designation>"
    }
  ]
}
```

### CatalogStandard shape

```json
{
  "catalog_version": "0.1",
  "publisher": "<canonical domain>",
  "designation": "<designation>",
  "title": "<short title>",
  "scope_statement": "<plain-language scope statement, not normative clause text>",
  "industry_sectors": [
    { "clasp_code": <int>, "label": "<label>", "isic_rev4": "<code>" }
  ],
  "licensing_uri": "<URI for the licensing flow>",
  "license_inquiries_uri": "<optional URI for human-mediated inquiries>",
  "editions": [
    {
      "edition": "<edition>",
      "pubdate": "<ISO-8601 date>",
      "is_current": <bool>,
      "canonical_language": "<BCP 47 tag>",
      "available_languages": ["<BCP 47 tag>", ...],
      "supersedes": ["<designation-edition>", ...],
      "superseded_by": "<designation-edition or null>",
      "regulatory_incorporations": [ /* RegulatoryIncorporation objects */ ],
      "table_of_contents": [
        { "clause": "<clause identifier>", "title": "<heading text>", "depth": <int>, "parent": "<parent clause identifier or null>" }
      ],
      "defined_terms": [
        { "term": "<term>", "clause": "<clause identifier>" }
      ]
    }
  ]
}
```

### Content rules

- `scope_statement` is a plain-language description. MUST NOT reproduce normative clause text.
- `table_of_contents` carries clause headings only. MUST NOT carry clause body text. A publisher MAY omit `table_of_contents`.
- Each `tocEntry` MAY carry an optional `parent` field referencing the clause identifier of its parent entry. Top-level entries omit `parent` (or set it to `null`). When `parent` is present, it MUST match the `clause` value of another entry in the same `table_of_contents` array.
- The `table_of_contents` array's order is normative. The publisher's chosen order is the canonical document order. Consumers MUST preserve array order when rendering or processing entries; consumers MUST NOT re-sort by `clause`, `title`, `depth`, `parent`, or any derived field. Sibling order is fully determined by array order.
- `defined_terms` carries (term, clause) only. MUST NOT carry definition text, descriptions, or paraphrases.

### HTTP semantics

Servers MUST emit `ETag` headers. Servers SHOULD emit `Last-Modified` and `Cache-Control` (with a reasonable `max-age`). Servers MUST support conditional GET (`If-None-Match` and `If-Modified-Since`); a server MUST return `304 Not Modified` when the AI System's conditional GET indicates the cached copy is current.

AI Systems MUST revalidate cached catalog responses via conditional GET before using them past the `Cache-Control` `max-age`. AI Systems SHOULD treat the `ETag` as opaque.

### What the catalog does not provide

- Normative clause text.
- Licensed content.
- A canonical aggregate. CLASP does not specify a directory that aggregates catalogs from multiple publishers (see [Open Issues](#open-issues)).

## Cross-publisher reference resolution

A CLASP-conformant Content Owner that receives a `resolve_reference` request for a target held by a different publisher MUST respond with a `mode = 5` (pointer) citation that includes the target publisher's catalog URI. The Content Owner MUST NOT return text from the other publisher's standard.

## Session revocation and envelope durability

A license issued by a Content Owner to an AI System MAY be revoked by the Content Owner before its natural expiration.

### Immediate effects

Upon revocation, the Content Owner MUST cause subsequent tool calls under the revoked `lid` to fail and MUST return an error response identifying the revocation. The Content Owner SHOULD make the revocation discoverable via `retrieval.ext.revocation_uri`.

### Durability of prior envelopes

Citation envelopes emitted prior to revocation remain durable and verifiable. The envelope's `content_hash` values are computed over the response bodies returned at retrieval time and do not change after revocation; the `retrieval_timestamp` field is load-bearing for determining whether the retrieval occurred before or after revocation.

This durability rule matches real-world engineering citation semantics: a Professional Engineer's stamp is not retroactively invalidated when a subscription later lapses.

### What revocation does not do

- Invalidate previously-emitted citation envelopes.
- Require the AI System to delete prior responses already emitted to deliverables.
- Prevent verification of prior `content_hash` values.
- Affect tool calls completed before the revocation timestamp.

## Multi-language behavior

CLASP treats an edition as one logical thing. Multiple-language renderings are renderings of the same edition.

The Content Owner declares `canonical_language` and `available_languages` in `scope.ext`. The AI System declares `preferred_languages` in `aisystemuse.ext`. The Content Owner returns the first available match, or `canonical_language` if no preference matches. The citation envelope's `language` field records the rendering returned.

### Canonical-language obligation for regulatory and stamped use

When the envelope's `language` is not `canonical_language` and `deliverable_type` is `stamped-deliverable` (3) or `submitted-to-regulator` (4), the AI System MUST either re-retrieve the cited clauses in the canonical language and attach a second envelope, or annotate the deliverable to identify the cited text as a translation of the canonical-language clause.

### Multi-language `compliance_check`

See [CLASP tool semantics: compliance_check, Multi-language findings](#multi-language-findings). One envelope per language; one `compliance_check` call per language.

## Deliverable type and session strictness

The `deliverable_type` declared in `aisystemuse.ext` is binding for the session:

1. The Content Owner authorizes the session against the declared `deliverable_type`. The Content Owner MAY require `exact_text_mode = 1` and `requester_authority = 0` for `stamped-deliverable` or `submitted-to-regulator`.
2. The citation envelope carries the declared `deliverable_type` unchanged.
3. The AI System MUST NOT emit content into a deliverable of higher strictness than the session's declared type. Strictness increase requires re-handshake.
4. The AI System MAY emit content into a deliverable of lower strictness without re-handshaking.

## Examples

Worked example exchanges are in [examples/](./examples/):

1. [01-design-consultation-search.md](./examples/01-design-consultation-search.md) — exploratory design consultation in a control-system IDE.
2. [02-cross-edition-regulatory.md](./examples/02-cross-edition-regulatory.md) — cross-edition comparison supporting a regulatory submission.
3. [03-compliance-check-stamped.md](./examples/03-compliance-check-stamped.md) — gap analysis for a PE-stamped deliverable. Demonstrates the `requester_sectors` field.
4. [bpec-catalog.md](./examples/bpec-catalog.md) — the BPEC discoverability catalog.

## Open Issues

Deferred until implementation experience produces concrete requirements:

1. **Session lifetime beyond revocation.** Clock-skew and mid-tool-call expiration edge cases.
2. **Multi-AI-System sessions.** Sessions where multiple AI Systems collaborate under a shared license, or where one AI System hands work to another.
3. **Aggregator catalogs.** A directory that walks multiple publisher catalogs. Arguably a separate concern from CLASP itself.

---

*This document is a working draft. Contributions and feedback welcome.*
