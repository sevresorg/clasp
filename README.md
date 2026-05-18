# CLASP

**Clause-Level Access and Standards Protocol.** An open specification that extends the IAB Tech Lab's [Content Monetization Protocols (CoMP)](https://github.com/IABTechLab/CoMP) v1.0 for the specific domain of engineering and technical standards.

CLASP is part of the [Sèvres](https://sevres.org) umbrella project. The specification is licensed CC-BY-4.0.

## What CLASP adds to CoMP

CoMP v1.0 defines a clean handshake between AI Systems and Content Owners, but its metadata vocabulary is shaped for media publishing: wordcount, transcript flags, video duration, image provenance. For engineering standards, the information that matters is different: clause identifiers, editions, normative references, defined terms, applicable industry sectors, and supersession history.

CLASP defines four things:

1. An `aisystemuse.ext` extension declaring engineering-specific intended uses, distinct from CoMP's publisher-oriented function taxonomy. Examples include design consultation, gap analysis against a spec, derivative standards work, and tooling integration.
2. A `scope.ext` extension declaring standards-specific metadata: publisher, designation, edition, clause-level addressability, normative references, defined terms, industry sector, and supersession chain.
3. A `retrieval.ext` extension declaring CLASP-conformant MCP server capabilities: clause retrieval, full-text and semantic search, defined-term resolution, edition-to-edition diff, and structured compliance check.
4. A free discoverability tier sitting outside CoMP, allowing agents to identify which standards exist and what their scope is before requesting a license. CoMP explicitly leaves discoverability out of scope.

## Relationship to CoMP

CLASP is strictly additive to CoMP. A CLASP-conformant Content Owner is also a CoMP-conformant Content Owner. A CoMP AI System that does not understand CLASP extensions will see a valid CoMP Package response and can fall back to the base retrieval interface; it will simply not benefit from the engineering-specific capabilities. See [docs/relationship-to-comp.md](./docs/relationship-to-comp.md).

## Status

Draft v0.1, current. See [CLASP-0.1.md](./CLASP-0.1.md). Worked examples and JSON Schemas are in [examples/](./examples/) and [schemas/](./schemas/).

Three issues are deferred until implementation experience produces concrete requirements: session lifetime beyond revocation, multi-AI-System sessions, and aggregator catalogs.

Note on naming: the standards-body names, designations, and publisher domains used in the spec and examples are fictional. The protocol is not specific to any of those bodies; the same shape applies to any real Content Owner adopting CLASP.

## Related repositories

CLASP is one of several sibling repos under [`sevresorg`](https://github.com/sevresorg):

- [`sevresorg/clasp-web`](https://github.com/sevresorg/clasp-web) — Nextra site at `clasp.sevres.org` that renders this spec. Carries this repo as a git submodule.
- [`sevresorg/norma`](https://github.com/sevresorg/norma) — Norma, the turnkey CLASP-conformant gateway platform for SDOs.
- [`sevresorg/clasp-mcp`](https://github.com/sevresorg/clasp-mcp) — open-source reference MCP server (planned).
- [`sevresorg/sevres-web`](https://github.com/sevresorg/sevres-web) — umbrella site at `sevres.org`.

## License

[Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/). See [LICENSE](./LICENSE).
