# CLASP schemas

JSON Schema (draft 2020-12) documents for the wire-level shapes defined in [CLASP-0.1](../CLASP-0.1.md). Implementers can use these to validate request bodies, response bodies, citation envelopes, and discoverability catalog documents.

## Schemas

| File | Purpose | Spec section |
| --- | --- | --- |
| [citation-envelope-0.1.json](./citation-envelope-0.1.json) | Envelope attached to every retrieval response. Encodes per-citation mode invariants (snippet object iff mode=2, diff object iff mode=3, pointer object iff mode=5, all-null otherwise). | Citation Envelope |
| [aisystemuse-ext-0.1.json](./aisystemuse-ext-0.1.json) | CLASP extension on the CoMP `AISystemUse.ext` field. Encodes the conditional that `target_clauses_numbering` requires `target_edition_compare`. | Extension: aisystemuse.ext |
| [scope-ext-0.1.json](./scope-ext-0.1.json) | CLASP extension on the CoMP `Scope.ext` field returned by Content Owners. | Extension: scope.ext |
| [retrieval-ext-0.1.json](./retrieval-ext-0.1.json) | CLASP extension on the CoMP `Retrieval.ext` field. Encodes the conditional that `max_artifact_bytes` is required when `compliance_check` is in `tools`. | Extension: retrieval.ext |
| [catalog-index-0.1.json](./catalog-index-0.1.json) | The thin CatalogIndex document served at `/.well-known/clasp-catalog`. | Discoverability Catalog |
| [catalog-standard-0.1.json](./catalog-standard-0.1.json) | The per-standard CatalogStandard document with editions, TOC (with `parent` field), and defined terms. | Discoverability Catalog |

All six schemas use JSON Schema draft 2020-12 (`$schema: "https://json-schema.org/draft/2020-12/schema"`).

## What's encoded versus what's prose-only

The schemas encode structural constraints (required fields, value enums, types, conditional dependencies that are mechanical). They do not encode:

- The "no normative text" rule on catalog documents. JSON Schema cannot distinguish a plain-language scope statement from normative clause text. This rule lives in the spec prose and is enforced by publisher policy.
- The canonical-language attestation obligation on stamped/regulator-bound deliverables.
- The hashing rules. JSON Schema validates the `content_hash` format (`sha256:` plus 64 hex chars) but not that the hash actually equals the content's SHA-256.
- The HTTP cache semantics for catalog responses.
- Regulator-jurisdiction filtering logic for multi-incorporation cases.
- The TOC array-order normative rule. JSON Schema preserves array order on parse but cannot prevent a consumer from reordering after parse.
- The `parent` field referential integrity: each non-null `parent` MUST match the `clause` of another entry in the same `table_of_contents`. JSON Schema cannot express cross-element references inside the same array. This is a consumer-side check.

## Versioning

Each schema embeds its `$id` as the canonical URL `https://clasp.example/schemas/<name>-0.1.json` and pins the version-bearing fields (`envelope_version`, `clasp_version`, `catalog_version`) to `"0.1"` as constants. Future spec revisions will publish parallel schemas rather than mutating existing ones.

Implementers SHOULD host these schemas at a stable URL on their own infrastructure and reference them from their `retrieval.ext.citation_envelope` field accordingly. The `clasp.example` URLs are reserved-for-documentation placeholders and do not resolve.

## Validation

Example with Python `jsonschema`:

```python
import json
from jsonschema import validate, Draft202012Validator

with open("citation-envelope-0.1.json") as f:
    schema = json.load(f)

with open("my-envelope.json") as f:
    instance = json.load(f)

Draft202012Validator.check_schema(schema)
validate(instance=instance, schema=schema)
```

The worked examples in [../examples/](../examples/) are valid against these schemas; they are the regression-test corpus.

## Not yet schematized

These remain prose-only in the spec:

- The MCP tool wire schemas (tool call arguments and response bodies, layer 2). These will be specified by Norma.
- The revocation list document format published at `retrieval.ext.revocation_uri`. CLASP-0.1 explicitly leaves the format to the Content Owner.
- Error response shapes.
