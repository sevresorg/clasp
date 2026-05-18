# CLASP examples

Worked example exchanges conformant with [CLASP-0.1](../CLASP-0.1.md). The three CoMP-handshake examples walk through the front-door handshake, the subsequent MCP tool calls, and the citation envelope attached to each tool response. The fourth example shows the discoverability catalog that sits in front of all of it.

The examples use the fictional standards-body names introduced in the CLASP-0.1 naming convention (PASC, TMF, BPEC, IESF). They are constructed to read plausibly without referring to any real engineering SDO.

| Example | Scenario | Tools / surface exercised | Notable surface |
| --- | --- | --- | --- |
| [01-design-consultation-search.md](./01-design-consultation-search.md) | Exploratory design consultation in a control-system IDE | `search`, `get_clause` | `mode = snippet` (2) and `mode = exact-text` (0); `target_clauses` omitted at session level; `individual-licensee`; single-language English |
| [02-cross-edition-regulatory.md](./02-cross-edition-regulatory.md) | Cross-edition comparison supporting a regulatory submission | `list_changes`, `resolve_reference` (same-publisher and cross-publisher) | `mode = diff` (3) and `mode = pointer` (5); `target_edition_compare` populated; `regulatory_incorporations` with transition-window fields; `language: null` on the pointer envelope; `available_languages: ["en", "fr"]` (only English requested) |
| [03-compliance-check-stamped.md](./03-compliance-check-stamped.md) | Cause-and-effect matrix gap analysis for a PE-stamped deliverable | `compliance_check`, `resolve_term` | `mode = structured-analysis` (4); artifact upload; `stamped-deliverable` with `individual-licensee`; `requester_sectors` populated; ISIC Rev 4 cross-reference annotated |
| [bpec-catalog.md](./bpec-catalog.md) | AI System follows the cross-publisher pointer from example 02 to the BPEC discoverability catalog | `CatalogIndex` and `CatalogStandard` | HTTP cache headers (`ETag`, `Last-Modified`, `Cache-Control`); TOC with `parent` field and normative array order; defined-terms without normative text; multi-edition renumbering case; non-English `available_languages` |

Together the four examples exercise every CLASP tool, every citation-envelope mode, every `requester_authority` value except `unknown`, four of five `deliverable_type` values, the discoverability catalog with `tocEntry.parent`, and the `requester_sectors` field.

## Reading order

Read 01 through 04 in order. Example 01 establishes the CoMP-handshake-then-MCP-tools layering most cleanly. Examples 02 and 03 assume that frame and focus on the citation envelope variants. Example 04 (the BPEC catalog) closes the loop with example 02's cross-publisher pointer, demonstrating the discoverability tier that sits in front of the CoMP handshake.

## What the examples do not show

- The `synthesized` citation mode (`mode = 1`). All examples use `exact-text` because all the sessions involve deliverables where verbatim quotation is preferred.
- The `unknown` `requester_authority` value (`2`).
- A fully non-English-canonical example. Example 02 lists French in `available_languages` and BPEC-200 lists German, but in both cases English is selected.
- License revocation in flight. CLASP-0.1 specifies revocation behavior in prose; no example shows a revocation event.
- `list_changes` with `target_clauses_numbering` explicitly set. Example 02 leaves `target_clauses` unset at session level; demonstrating the override field would require restructuring the scenario.
- Multi-language `compliance_check`. Example 03 declares one language; the multi-language pattern would require an English-plus-Japanese (or similar) deliverable.
- Multi-AI-System sessions and other lifecycle edge cases.
- An aggregator that walks multiple publisher catalogs. CLASP intentionally does not specify aggregators.
