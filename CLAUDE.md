# CLASP — repo context

This is `sevresorg/clasp`: the canonical home of the **CLASP (Clause-Level Access and Standards Protocol)** specification, schemas, and worked examples. CLASP is part of the [Sèvres](https://sevres.org) umbrella project.

This document is the per-repo context-loading brief. Read it in full before responding to other instructions.

---

## 1. What this repo is

CLASP is an open specification that extends the IAB Tech Lab's Content Monetization Protocols (CoMP) v1.0 for the specific domain of engineering and technical standards. It is licensed CC-BY-4.0.

The repo contains:

- `CLASP-0.1.md` — the current working draft of the specification.
- `schemas/` — JSON Schema 2020-12 documents for the wire-level shapes (`citation-envelope`, `aisystemuse-ext`, `scope-ext`, `retrieval-ext`, `catalog-index`, `catalog-standard`).
- `examples/` — full request/response exchange examples using fictional standards bodies (PASC, TMF, BPEC, IESF).

## 2. The broader Sèvres project

CLASP is one of several sibling repos under [`sevresorg`](https://github.com/sevresorg):

- **`sevresorg/clasp-web`** — the Nextra site at `clasp.sevres.org` that renders this spec. Carries this repo as a git submodule.
- **`sevresorg/norma`** — Norma, the turnkey CLASP-conformant gateway platform for SDOs (a product, not part of the open spec). Serves `norma.sevres.org`.
- **`sevresorg/clasp-mcp`** — the open-source reference MCP server implementing the CLASP tool surfaces (future, currently a stub).
- **`sevresorg/sevres-web`** — the umbrella site at `sevres.org`.

## 3. Problem this spec exists to solve

Engineering Standards Developing Organizations (SDOs) have responded to the rise of agentic AI by either prohibiting AI ingestion of their standards outright or restricting it through license terms. ASTM, ISA, IEC, IEEE, ASHRAE, and others have published explicit prohibitions. The publishing industry has moved past this posture: the IAB Tech Lab finalized CoMP v1.0 on April 28, 2026.

CLASP demonstrates that an SDO can preserve its subscription revenue and IP protection while making content accessible to licensed AI agents. The architecture is strictly additive to existing SDO infrastructure: existing subscription platforms continue to serve human users; new CLASP and CoMP endpoints serve licensed AI agents.

## 4. Architectural decisions already made

These are settled. Do not reopen them unless explicitly asked.

**4.1 CLASP sits on CoMP.** CoMP v1.0 is adopted as the front-door handshake protocol without modification. CLASP populates the `ext` placeholders.

**4.2 MCP is the primary retrieval format.** CoMP recognizes MCP as a first-class content-delivery format. The CLASP retrieval layer is an MCP server exposing six standards-specific tools.

**4.3 CLASP extends CoMP via three `ext` fields:** `aisystemuse.ext`, `scope.ext`, `retrieval.ext`.

**4.4 Discoverability is adjacent to CoMP.** A free, unauthenticated catalog at `/.well-known/clasp-catalog`.

**4.5 Citation envelope is mandatory.** Every clause retrieved through a CLASP endpoint must carry a structured citation envelope.

**4.6 Auth is OAuth2. Wire format is JSON over HTTPS.**

## 5. About the maintainer

Andy Robinson, ConSynSys, biotech/pharma industrial automation. Comfortable with industrial automation domain (DCS, SIS, PLC, ISA-88, ISA-95), Python, Go, Docker/Kubernetes. Chemical engineering background.

## 6. How to work in this repo

**6.1 Communication.** Direct, technical, concise. Sophisticated technical reader. No padding, no marketing tone.

**6.2 No em dashes.** Hard rule. Use semicolons, commas, parentheses, or shorter sentences.

**6.3 Editing.** When editing existing writing, tighten only and preserve tone. Do not soften assertions.

**6.4 Specification tone.** Match the register of the CoMP specification: precise, declarative, no hype, no jargon that doesn't earn its keep.

**6.5 Be opinionated when it serves the work.** Push back when push-back is warranted.

**6.6 Don't invent normative facts about real standards.** ISA-88 is real; its clause numbers are not in your training data with granularity to fabricate. Use the fictional example standards (PASC, TMF, BPEC, IESF) or flag placeholders explicitly.

**6.7 File naming.**
- Specifications: title-case with version suffix, `CLASP-0.1.md`, `CLASP-0.2.md`. Current draft is the most recent number; superseded drafts stay in the repo for traceability when there are real revisions.
- Documentation: lowercase-hyphenated, `relationship-to-comp.md`.
- Examples: numbered and named after the scenario, `01-design-consultation-search.md`.

**6.8 Markdown style.**
- ATX headings (`#`, `##`), not Setext.
- Sentence-case section titles.
- Inline code with backticks for field names, designations, identifiers.
- Tables for structured field definitions, matching CoMP's style.
- Prefer prose over bullets for explanatory content.

## 7. Open architectural questions

These are live:

- Industry-sector taxonomy alignment (currently a starter list with ISIC Rev 4 cross-references; alignment with NAICS, NACE, or GICS deferred).
- Whether `engineering_intent` in `aisystemuse.ext` should replace or be combined with CoMP's `function` taxonomy.
- Versioning and supersession semantics for incorporation-by-reference into regulations.
- Whether CLASP should eventually be proposed back to the IAB Tech Lab as an official CoMP profile.
- Session lifetime beyond revocation; multi-AI-System sessions; aggregator catalogs (deferred until implementation experience produces concrete requirements).

## Git Push

**GitHub push user:** `arobinsongit`

```bash
RAW_USER="arobinsongit"
PREV_USER=$(gh api user --jq .login 2>/dev/null)
if [ "$PREV_USER" != "$RAW_USER" ]; then gh auth switch --user "$RAW_USER"; fi
git push origin <branch>
PUSH_RC=$?
if [ "$PREV_USER" != "$RAW_USER" ] && [ -n "$PREV_USER" ]; then gh auth switch --user "$PREV_USER"; fi
exit $PUSH_RC
```
