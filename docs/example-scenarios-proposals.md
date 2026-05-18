# CLASP example scenarios: proposals for review

Status: draft for Andy's review, May 2026. Not normative. This document proposes three candidate exchange scenarios for `clasp/examples/` and lists v0.1 fields that the scenarios surfaced as candidates for change.

The goal of this step is to stress the CLASP v0.1 `ext` fields against concrete cases, surface ambiguities, and decide which two or three scenarios to develop into full request/response pairs.

All three scenarios use fictional standard designations chosen to look domain-plausible while clearly not colliding with real designations. Each is flagged as fictional in the proposal.

---

## Scenario A: Exploratory design consultation in a control-system IDE

A control engineer at a CDMO is drafting a recipe for a new perfusion-fed fermentation skid in a vendor-supplied control-system IDE that has an embedded AI assistant. The recipe must reconcile classic batch unit procedure decomposition with continuous-feed material balance; the engineer doesn't know the standard well and asks the assistant "what does the current standard say about phase transitions between batch unit operations and continuous feed states?"

The AI System issues a `search` against the engineer's licensed scope for "phase transition AND continuous feed", reads the result snippets, picks one or two hits, and follows with `get_clause` calls to retrieve the full text of the most relevant clauses. The engineer is reading the result conversationally and may paste the retrieved clauses into an internal design memo; nothing is being externally distributed or stamped.

Fictional standard: `ISA-188.01-2025, Hybrid Batch-Continuous Process Control, Part 1: Models and Terminology`. (Fictional; ISA-188 is not a real ISA committee series. Industry sector: bioprocess.)

What this demonstrates that the others won't: the `search` to `get_clause` chain, including how the citation envelope handles snippet results from `search` differently from full-text results from `get_clause`; an `exploratory` to `internal-document` deliverable type where `exact-text` mode is not strictly required; how `scope.ext.publisher` and `scope.ext.edition` carry edition pinning when the engineer didn't know what edition was current. CoMP framing: `function = [3]` (ai-input), `subfn = [1]` (rag), `resdis = 1`.

---

## Scenario B: Cross-edition comparison supporting a continuous-manufacturing regulatory submission

A process engineer at a small-molecule sponsor is preparing a CMC supplement for an FDA submission switching a tablet line from batch to continuous direct compression. The relevant verification-of-continuous-manufacturing standard has just gone through a major edition revision; the firm's existing validation procedures were qualified against the prior edition, and the agency's reviewer guidance was last updated referencing the prior edition by name. The engineer needs a structured, citable diff between the two editions to support the supplement's justification narrative and to identify which qualified procedures need re-evaluation.

The AI System, embedded in the sponsor's regulatory-authoring environment, issues `list_changes` between the two editions of the standard, then follows specific normative references in the new edition via `resolve_reference`. At least one of those references points to a different SDO's document that the sponsor does not hold a license to (this is the interesting case); the response should return a structured pointer rather than text. The output will be quoted in a document submitted to FDA.

Fictional standard: `ASTM E3050-2023` versus `ASTM E3050-2026, Standard Guide for Verification of Continuous Pharmaceutical Manufacturing Systems`. (E3050 is invented; chosen to read like an extension of the real E2500 lineage. Industry sectors: pharmaceutical, bioprocess.)

What this demonstrates that the others won't: cross-edition scoping (the `target_edition` field as written is singular, but `list_changes` inherently has a "from" and a "to"); how `supersedes` and `superseded_by` populate when the engineer is documenting the supersession itself; `submitted-to-regulator` deliverable forcing `exact_text_mode = 1` and the strictest citation envelope semantics; `resolve_reference` returning a discoverability pointer for a target held by another Content Owner. CoMP framing: `function = [3]` (ai-input), `subfn = [2]` (grounding), `resdis = 1`, `package.citation = 1` strongly recommended.

---

## Scenario C: Cause-and-effect matrix gap analysis against a safety standard, for a PE-stamped deliverable

A process safety engineer at an EPC is reviewing a draft cause-and-effect (C&E) matrix for the SIS of a hydroprocessing unit before issuing the matrix as a stamped deliverable. The C&E matrix has been authored partly by hand and partly by an upstream tool; the engineer wants an independent gap analysis against the specific conformance clauses of the relevant industry standard before applying their PE seal. The engineer's workstation runs an AI agent that already holds a license to the standard.

The AI System submits the draft C&E matrix to `compliance_check` against an explicit clause set (the conformance clauses of the standard plus a normative annex); the response is a structured gap analysis where each finding is anchored to a specific clause. The agent also issues `resolve_term` for one ambiguous defined term ("safe state") that appears in the matrix to confirm the engineer's interpretation matches the standard's vocabulary clause. The result is incorporated into the stamped deliverable record.

Fictional standard: `ISA-184.02-2025, Cause-and-Effect Matrix Conformance for Safety Instrumented Systems`. (ISA-184 is invented; chosen adjacent to the real ISA-84 / IEC 61511 family without colliding. Industry sectors: oil-and-gas, chemical.)

What this demonstrates that the others won't: the `compliance_check` tool, which is the most architecturally novel CLASP tool because it takes an artifact as input and returns structured analysis rather than text; `resolve_term`; how the citation envelope wraps a structured response that cites many clauses at once rather than returning one clause; `stamped-deliverable` deliverable type forcing `exact_text_mode` for any quoted clause text inside findings; CoMP `agent-actions` sub-function (`subfn = [4]`), which Scenarios A and B don't touch. CoMP framing: `function = [1]` (ai-all), `subfn = [4]` (agent-actions), `resdis = 1`.

---

## Coverage matrix

| Dimension | Scenario A | Scenario B | Scenario C |
| --- | --- | --- | --- |
| `engineering_intent` | design-consultation (0) | regulatory-submission (5) | gap-analysis (1) |
| Target scope shape | whole standard, then specific clauses after search | cross-edition (two editions) | specific clauses plus annex |
| `deliverable_type` | internal-document (1) | submitted-to-regulator (4) | stamped-deliverable (3) |
| Primary CLASP tools | `search`, `get_clause` | `list_changes`, `resolve_reference` | `compliance_check`, `resolve_term` |
| CoMP `function` | 3 (ai-input) | 3 (ai-input) | 1 (ai-all) |
| CoMP `subfn` | 1 (rag) | 2 (grounding) | 4 (agent-actions) |
| Industry sectors | bioprocess | pharmaceutical, bioprocess | oil-and-gas, chemical |

Each pair of scenarios differs on at least three of the four dimensions the brief named (intent, scope, deliverable, tool). All six listed CLASP tools (`get_clause`, `search`, `resolve_term`, `list_changes`, `resolve_reference`, `compliance_check`) are exercised across the three.

---

## v0.1 fields and gaps that already look like change candidates

These came out of writing the proposals; some are concrete field changes, some are structural clarifications.

1. **`target_edition` is singular, but `list_changes` needs two.** Either add `target_edition_from` / `target_edition_to`, or generalize to `target_editions` as an ordered array, or define a tool-specific request shape for `list_changes`. Scenario B forces the issue.

2. **Where the search query lives.** Scenario A's `search` call needs a query string. CLASP's `aisystemuse.ext` does not currently carry one. The likely correct architecture is that queries are MCP tool arguments after handshake, and the CoMP-layer handshake does not see them. The spec should say this explicitly; right now it is implicit and a reader could easily think CLASP wants the query in `aisystemuse.ext`.

3. **Citation envelope is single-clause-shaped.** The v0.1 sketch lists "clause identifier" (singular), "content hash of the returned clause text" (singular), and a binary `exact-text` vs `synthesized` mode. The three scenarios produce four distinct response shapes the envelope has to wrap: a single clause (A's `get_clause`), a snippet hit list (A's `search`), a structured edition diff (B), and a multi-clause gap analysis (C). The mode indicator needs at least: `exact-text`, `synthesized`, `snippet`, `diff`, `structured-analysis`. The envelope probably needs to nest: one outer envelope per response, with one inner citation per cited clause.

4. **`compliance_check` semantics are essentially undefined.** v0.1 lists the tool but does not say how the artifact reaches the server (MIME type, size, schema), what the response shape is, or how the citation envelope wraps each finding. Norma can specify the wire format, but CLASP itself needs at least a paragraph naming the response shape and the citation discipline (each finding MUST carry an envelope for any clause it cites).

5. **`resolve_reference` across Content Owners.** Scenario B's interesting case: the normative reference points to a standard held by a different SDO. The current spec is silent on this. We should specify that when the referenced target is not in the responding Content Owner's licensed scope, the response is a structured pointer (publisher, designation, edition, clause, and discoverability URI) rather than text. This is also a natural place to tie the discoverability tier into the retrieval flow.

6. **`target_clauses` form.** v0.1 says clause identifiers are hierarchical and gives `5.2.3` and `Annex C, C.4` as examples, but doesn't say how to express ranges (e.g., "7.2 through 7.4"), annex-plus-numbered-subclause syntax, or whether the server is allowed to reject malformed strings versus best-effort-resolve them. The `clause_scheme` URI is the right hook but the spec doesn't say it is normative for parsing.

7. **`deliverable_type` is single-valued and not re-declarable.** A real session can start exploratory and end up stamped. The spec should say either: re-declaration is required when the value increases (in strictness), or the strictest value declared during the session is binding for the citation envelope, or it is the AI System's responsibility to refuse downstream emission if the envelope's recorded `deliverable_type` is weaker than the actual use.

8. **Authorial provenance for stamped or submitted work.** Open issue #5 in v0.1. Scenarios B and C both involve work product that has legal weight; whether the license is held by the individual PE or by the employing firm matters for liability and for how the license fingerprint in the citation envelope is interpreted. Worth deciding before v0.2.

9. **Industry sector semantics.** `industry_sectors` is on `scope.ext` (i.e., the Content Owner declares which sectors the standard applies to). It is not on `aisystemuse.ext`, so the AI System cannot declare the requester's sector. That asymmetry may be intentional, but Scenario C (a safety standard that applies to several sectors) suggests the requester's sector is occasionally relevant for `compliance_check`, because some clauses are sector-conditional. Worth a design call.

10. **CoMP `function` versus `engineering_intent`.** v0.1 explicitly leaves this as an open issue. Writing the three scenarios, the natural pattern was to use `function` for the CoMP-layer commercial classification and `engineering_intent` for the CLASP-layer workflow signal, and they did not fight each other. Tentative recommendation: keep them union'd as the draft does; do not replace `function`. Worth confirming before freezing.

---

## Decisions wanted before drafting full payloads

- Which two or three of A, B, C to develop into full request/response pairs in `clasp/examples/`?
- For any v0.1 gap above that we want to fix before baking examples in, patch the spec first (v0.1.1) so the examples match a coherent draft.
- For the `target_edition` issue in particular (item 1): pick a representation now, because Scenario B's payload depends on it.
- For the citation envelope shape (item 3): decide whether v0.1.1 specifies the envelope normatively, or whether the examples are allowed to forward-reference a v0.2 envelope schema.
