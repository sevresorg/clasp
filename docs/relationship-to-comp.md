# Relationship to CoMP

## Summary

CLASP is an extension profile for the IAB Tech Lab Content Monetization Protocols (CoMP) v1.0 specification. CLASP is strictly additive: every CLASP exchange is a valid CoMP exchange. CoMP-only AI Systems can interoperate with CLASP-conformant Content Owners; they will simply not benefit from the engineering-specific capabilities that CLASP defines.

## Why we adopted CoMP rather than inventing

CoMP was finalized on April 28, 2026, as the output of a multi-stakeholder working group convened by a non-profit standards-setting consortium with a track record of producing widely-adopted open specifications (OpenRTB, ads.txt, VAST, Open Measurement SDK). The IAB Tech Lab pattern is to define interchange-layer protocols, license them under Creative Commons, and let an ecosystem of independent implementers build products on top. CLASP follows the same pattern, one rung up the stack.

Inventing a new front-door handshake for engineering standards specifically would fragment the emerging ecosystem and would force AI System implementers to support multiple parallel handshake protocols. Adopting CoMP and extending it via the `ext` mechanism that CoMP provides on every object lets engineering SDOs participate in a single growing ecosystem of CoMP-aware AI Systems.

## What CoMP provides

CoMP defines:

- An `AISystem` request object: identifies the AI System making the request, its user agent, its registered ID, and the intended use of the requested content.
- An `AISystemUse` object: declares the license ID claimed by the AI System, the authentication method, the URI or scope of content requested, and the function and sub-function the AI System will perform with the content.
- A `Package` response object: returned by the Content Owner, describing the offered content, the licensing URL, the reporting URL, the citation requirement, the retrieval endpoint, and the authentication required at retrieval time.
- A `Scope` object inside Package: describes the content offered in terms of content types (text, video, image, audio), pricing model, allowed use, and per-asset metadata.
- A `Retrieval` object inside Package: specifies the endpoint URI, the authentication type (none, api_key, oauth2, SSL), and the delivery format (HTML, RSS, API, MCP, NLWeb, XML, NewsML).
- An `ext` placeholder on every object, explicitly defined as a placeholder for implementer-specific extensions.

CoMP recognizes MCP as a first-class delivery format. Example 8 of the CoMP specification shows a Package response with `retrieval.type` set to 3 (MCP) and `retrieval.endpoint` set to an MCP connection URI.

## What CoMP does not provide

CoMP is explicit about its scope boundaries. The following are not in scope and are left to implementers:

- Bot blocking strategy
- Commercial licensing terms
- Clearing-house functionality, including token issuance and payment
- Supply discovery
- Reporting mechanics

For engineering standards specifically, CoMP also does not provide:

- A vocabulary for engineering-specific intended uses (design consultation, gap analysis, derivative work, regulatory submission)
- A metadata schema for standards-specific content (designation, edition, clause addressability, normative references, defined terms, industry sector, supersession)
- A capability declaration for MCP servers that exposes standards-specific tools (clause retrieval, edition diff, defined-term resolution)
- A discoverability mechanism for unlicensed metadata browsing
- A citation envelope that downstream deliverables can attach to retrieved clauses

These are the gaps CLASP fills.

## How CLASP extends CoMP

CLASP populates three `ext` fields with engineering-standards vocabulary:

| CoMP field | CLASP extension purpose |
| --- | --- |
| `aisystemuse.ext` | Engineering-specific intended uses, target designation/edition/clauses, deliverable type |
| `scope.ext` | Standards-specific content metadata: publisher, designation, edition, supersession, normative refs, industry sector |
| `retrieval.ext` | CLASP-conformant MCP tool capability declaration, exact-text mode support, citation envelope schema URI |

CLASP additionally specifies a discoverability tier that sits adjacent to CoMP, not within it. This tier exposes a `/.well-known/clasp-catalog` endpoint conventionally reachable without authentication, providing the metadata an AI System needs to recognize that a relevant standard exists before requesting a license.

## Forward compatibility

CLASP is versioned independently of CoMP. The `clasp_version` field is required in every CLASP extension object and allows CLASP-aware AI Systems to detect the profile version at the front-door handshake. CoMP v1.0 is final and stable; future CoMP revisions will be tracked by CLASP as needed.

## Upstream contribution

It is an open question whether CLASP should be proposed back to the IAB Tech Lab as an official CoMP profile, published as an independent specification that references CoMP, or both. The IAB Tech Lab IPR Policy and the Creative Commons licensing of CoMP both permit independent extension specifications. The decision will likely depend on which engineering SDOs express interest first and whether they prefer a single integrated specification or an extension profile they can adopt independently.
