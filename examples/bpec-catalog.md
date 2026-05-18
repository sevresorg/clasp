# Example: BPEC discoverability catalog

**Scenario:** an AI System following a cross-publisher reference pointer (from [example 02, Step 5](./02-cross-edition-regulatory.md)) lands on the BPEC discoverability catalog. The pointer told it that `BPEC-200:2024` clause `8.3.2` exists; the catalog tells it what BPEC is, what `BPEC-200` covers, and how to license it.

**CLASP version:** `0.1`.

**Authentication:** none. The catalog is the free, unauthenticated discoverability tier.

**Cache semantics:** HTTP. Both endpoints emit `ETag` and `Last-Modified`. AI Systems revalidate with `If-None-Match`.

---

## Step 1: AI System fetches the catalog index

The cross-publisher pointer from example 02 carried `discoverability_uri = "https://bpec.example/.well-known/clasp-catalog"`. The AI System issues an unauthenticated GET against that URL.

**Request:**

```
GET /.well-known/clasp-catalog HTTP/1.1
Host: bpec.example
Accept: application/json
```

**Response:**

```
HTTP/1.1 200 OK
Content-Type: application/json
ETag: "ix-2026-04-18-v7"
Last-Modified: Sat, 18 Apr 2026 09:14:22 GMT
Cache-Control: public, max-age=3600

{
  "catalog_version": "0.1",
  "publisher": "bpec.example",
  "publisher_name": "Bioprocess and Pharmaceutical Equipment Council",
  "publisher_licensing_overview_uri": "https://bpec.example/licensing",
  "standards": [
    {
      "designation": "BPEC-100",
      "title": "Hygienic Design of Bioprocess Vessels",
      "current_edition": "2024",
      "catalog_uri": "https://bpec.example/.well-known/clasp-catalog/BPEC-100"
    },
    {
      "designation": "BPEC-200",
      "title": "Hygienic Design of Single-Use Bioprocess Components",
      "current_edition": "2024",
      "catalog_uri": "https://bpec.example/.well-known/clasp-catalog/BPEC-200"
    },
    {
      "designation": "BPEC-300",
      "title": "Cleanability and Sterilizability of Bioprocess Equipment",
      "current_edition": "2023",
      "catalog_uri": "https://bpec.example/.well-known/clasp-catalog/BPEC-300"
    },
    {
      "designation": "BPEC-410",
      "title": "Material Compatibility for Pharmaceutical Process Contact Surfaces",
      "current_edition": "2025",
      "catalog_uri": "https://bpec.example/.well-known/clasp-catalog/BPEC-410"
    }
  ]
}
```

### What this demonstrates

- The index is intentionally thin. Each entry has just enough to let the AI System decide which per-standard document to fetch next.
- `current_edition` flags the publisher's current edition for each designation. Older editions are visible only in the per-standard `editions` array.
- The catalog index does not include scope statements, defined terms, or supersession chains. Those live in the per-standard `CatalogStandard` documents.
- The `ETag` is opaque to the AI System; on revalidation, the AI System simply echoes it back in `If-None-Match`.

---

## Step 2: AI System fetches the per-standard document

The pointer from example 02 named `BPEC-200`. The AI System follows `catalog_uri` from the index entry for that designation.

**Request:**

```
GET /.well-known/clasp-catalog/BPEC-200 HTTP/1.1
Host: bpec.example
Accept: application/json
```

**Response (abbreviated to one edition and a representative TOC subset for readability):**

```
HTTP/1.1 200 OK
Content-Type: application/json
ETag: "bpec200-2024-rev3"
Last-Modified: Fri, 19 Jan 2026 16:02:11 GMT
Cache-Control: public, max-age=3600

{
  "catalog_version": "0.1",
  "publisher": "bpec.example",
  "designation": "BPEC-200",
  "title": "Hygienic Design of Single-Use Bioprocess Components",
  "scope_statement": "This standard establishes hygienic design requirements for single-use bioprocess components in pharmaceutical and biotechnology manufacturing, including wetted-path materials, joints and connections, extractables and leachables qualification, and cleanability of any reusable interfaces. The standard applies to components in contact with active pharmaceutical ingredient streams, intermediate process streams, and clean utility streams. It does not address sterilization validation, which is covered by BPEC-300.",
  "industry_sectors": [
    { "clasp_code": 0, "label": "bioprocess", "isic_rev4": "C21" },
    { "clasp_code": 1, "label": "pharmaceutical", "isic_rev4": "C21" }
  ],
  "licensing_uri": "https://bpec.example/licensing/BPEC-200",
  "license_inquiries_uri": "https://bpec.example/licensing/contact",
  "editions": [
    {
      "edition": "2024",
      "pubdate": "2024-06-12",
      "is_current": true,
      "canonical_language": "en",
      "available_languages": ["en", "de"],
      "supersedes": ["BPEC-200-2019"],
      "superseded_by": null,
      "regulatory_incorporations": [],
      "table_of_contents": [
        { "clause": "1",       "title": "Scope",                                              "depth": 1 },
        { "clause": "2",       "title": "Normative references",                               "depth": 1 },
        { "clause": "3",       "title": "Terms and definitions",                              "depth": 1 },
        { "clause": "4",       "title": "General hygienic design principles",                 "depth": 1 },
        { "clause": "5",       "title": "Wetted-path materials",                              "depth": 1 },
        { "clause": "5.1",     "title": "Polymer selection",                                  "depth": 2, "parent": "5"   },
        { "clause": "5.2",     "title": "Elastomer selection",                                "depth": 2, "parent": "5"   },
        { "clause": "5.3",     "title": "Material qualification documentation",               "depth": 2, "parent": "5"   },
        { "clause": "6",       "title": "Joints and connections",                             "depth": 1 },
        { "clause": "7",       "title": "Extractables and leachables qualification",          "depth": 1 },
        { "clause": "8",       "title": "Active pharmaceutical ingredient stream contact",    "depth": 1 },
        { "clause": "8.1",     "title": "General requirements",                               "depth": 2, "parent": "8"   },
        { "clause": "8.2",     "title": "Component qualification",                            "depth": 2, "parent": "8"   },
        { "clause": "8.3",     "title": "Process contact surfaces",                           "depth": 2, "parent": "8"   },
        { "clause": "8.3.1",   "title": "Surface finish requirements",                        "depth": 3, "parent": "8.3" },
        { "clause": "8.3.2",   "title": "Conformance criteria for direct API contact",        "depth": 3, "parent": "8.3" },
        { "clause": "8.3.3",   "title": "Documentation requirements",                         "depth": 3, "parent": "8.3" },
        { "clause": "9",       "title": "Cleanability of reusable interfaces",                "depth": 1 },
        { "clause": "Annex A", "title": "Worked examples of polymer selection rationale",     "depth": 1 },
        { "clause": "Annex B", "title": "Reference extractables test methods",                "depth": 1 }
      ],
      "defined_terms": [
        { "term": "single-use component",            "clause": "3.1.2"  },
        { "term": "wetted path",                     "clause": "3.1.5"  },
        { "term": "process contact surface",         "clause": "3.1.7"  },
        { "term": "API stream",                      "clause": "3.1.9"  },
        { "term": "extractables",                    "clause": "3.1.14" },
        { "term": "leachables",                      "clause": "3.1.15" },
        { "term": "conformance assessment",          "clause": "3.1.21" }
      ]
    },
    {
      "edition": "2019",
      "pubdate": "2019-08-30",
      "is_current": false,
      "canonical_language": "en",
      "available_languages": ["en"],
      "supersedes": [],
      "superseded_by": "BPEC-200-2024",
      "regulatory_incorporations": [],
      "table_of_contents": [
        { "clause": "1",   "title": "Scope",                            "depth": 1 },
        { "clause": "2",   "title": "Normative references",             "depth": 1 },
        { "clause": "3",   "title": "Terms and definitions",            "depth": 1 },
        { "clause": "4",   "title": "Materials",                        "depth": 1 },
        { "clause": "5",   "title": "Components",                       "depth": 1 },
        { "clause": "6",   "title": "Connections",                      "depth": 1 }
      ],
      "defined_terms": [
        { "term": "single-use component",  "clause": "3.1.1" },
        { "term": "wetted surface",        "clause": "3.1.3" }
      ]
    }
  ]
}
```

### What this demonstrates

- The `scope_statement` is a plain-language description, not normative clause text. It tells the AI System what the standard covers without quoting it.
- The `table_of_contents` carries clause identifiers, heading titles, a depth integer for visual nesting, and an explicit `parent` field pointing to the clause identifier of the entry's parent. Top-level entries omit `parent`. The AI System can build a tree by following `parent` pointers (`8.3.2` -> `8.3` -> `8` -> top-level); array order is normative and determines sibling order. The pointer from example 02 named clause `8.3.2`; the AI System can now report to the engineer that this clause is titled "Conformance criteria for direct API contact", that its direct parent is `8.3` ("Process contact surfaces"), and that the ancestor chain terminates at section `8` ("Active pharmaceutical ingredient stream contact").
- The `defined_terms` array exposes term names and the clauses where they are defined. No definition text. An engineer who needs to know what `process contact surface` means is directed by the AI System to the BPEC-200 licensing flow.
- The `editions` array includes the superseded `2019` edition with its own (smaller) TOC and defined-terms list. Clause numbering differs between editions: the term `single-use component` is at `3.1.1` in 2019 and `3.1.2` in 2024. This is the renumbering case that the protocol's `target_clauses` field has open across editions.
- `available_languages: ["en", "de"]` declares that BPEC-200:2024 has an English and a German rendering. An AI System with `preferred_languages: ["de", "en"]` would receive the German rendering when it later licenses the standard.
- `regulatory_incorporations` is empty for BPEC-200; this standard is not currently incorporated by reference into any regulator's published rules.

---

## Step 3: AI System routes the engineer to licensing

Based on what the catalog returned, the AI System can now tell the engineer:

> The 2026 edition of TMF-3050 (the standard you're working with) normatively cites BPEC-200:2024 clause 8.3.2, which is titled "Conformance criteria for direct API contact" and sits under section 8 ("Active pharmaceutical ingredient stream contact"). BPEC-200 is published by the Bioprocess and Pharmaceutical Equipment Council and is available in English and German. To retrieve the actual clause text, you need a BPEC license; the licensing flow is at https://bpec.example/licensing/BPEC-200.

None of that summary required exposing any normative clause text from BPEC-200. The AI System has used only catalog metadata.

---

## What this example demonstrates that the others do not

- The discoverability catalog itself (CatalogIndex and CatalogStandard).
- HTTP-layer caching semantics (`ETag`, `Last-Modified`, `Cache-Control`).
- The cross-publisher resolution path from the [example 02 pointer](./02-cross-edition-regulatory.md) to the actual catalog data the AI System uses to route the engineer.
- The "no normative text" IP boundary: heading titles and defined-term names cross it cleanly; definition text and clause body text do not.
- Multi-edition representation in the catalog, including the renumbering case (`3.1.1` versus `3.1.2` for `single-use component`).
- A non-English `available_languages` entry (German).

## Notes on the protocol surface

- The `Cache-Control: max-age=3600` here is illustrative. Publishers SHOULD pick a value that reflects how often their catalog actually changes; for most publishers, weekly or daily would be reasonable.
- The `ETag` value is opaque. The example uses human-readable strings (`"ix-2026-04-18-v7"`, `"bpec200-2024-rev3"`) for illustration; production deployments commonly use hashes.
- BPEC-200's German rendering (`available_languages: ["en", "de"]`) is illustrative; the catalog cannot tell the AI System the German clause headings without exposing the German TOC. v0.2.2 does not specify whether `table_of_contents` should be returned in a requested language; in practice publishers would return the canonical-language TOC by default and may or may not offer language-tagged TOC variants. This is a small spec gap worth noting.
- The catalog has no authentication, so a publisher concerned about scraping for competitive intelligence will need CDN-level rate limiting or other operational controls. The protocol does not specify these.
