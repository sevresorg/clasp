# Example 01: Design consultation with search and get_clause

**Scenario:** an exploratory design consultation in a control-system IDE that hosts an embedded AI assistant. The engineer is drafting a recipe for a new perfusion-fed fermentation skid and needs to reconcile classic batch unit procedure decomposition with continuous-feed material balance.

**Tools exercised:** `search`, `get_clause`.

**Standard (fictional):** `PASC-188.01-2025, Hybrid Batch-Continuous Process Control, Part 1: Models and Terminology`.

**Engineering intent:** `design-consultation` (0).

**Deliverable type:** `internal-document` (1).

**Requester authority:** `individual-licensee` (0). The license is held by the individual engineer through a CDMO-provided subscription.

**CLASP version:** `0.1`.

**Language:** canonical `en`; AI System declares `preferred_languages: ["en"]`.

---

## Engineering context

A control engineer at a contract development and manufacturing organization is writing the unit recipe for a new bioreactor in the vendor's control-system IDE. The IDE has a CLASP-aware AI assistant. The engineer asks: "what does the current standard say about phase transitions between batch unit operations and continuous feed states?"

The assistant has no prior knowledge of which clauses are relevant. It issues a `search` to find candidates, then a `get_clause` to retrieve the most relevant clause verbatim. The engineer reads the result conversationally and may paste the retrieved clause into an internal design memo. Nothing is being externally distributed or stamped.

---

## Step 1: CoMP handshake request

The AI System makes a single CoMP request declaring the session's intended scope. It does not include the search query at this layer; queries are MCP tool arguments and are exchanged in Step 3.

```json
{
  "aisystem": {
    "name": "control-ide-assistant.example",
    "ua": "ControlIDE-AI/1.4",
    "id": "TL-REG-771122",
    "aisysuse": {
      "lid": "PASC-LIC-CDMO-2026-44219",
      "aiauth": 2,
      "uri": ["https://retrieval.pasc.example/mcp"],
      "scope": 5,
      "function": [3],
      "subfn": [1],
      "resdis": 1,
      "ext": {
        "clasp_version": "0.1",
        "engineering_intent": [0],
        "target_designation": ["PASC-188.01"],
        "target_edition": "2025",
        "preferred_languages": ["en"],
        "deliverable_type": 1,
        "requester_authority": 0
      }
    }
  }
}
```

### What this declares

- `function = [3]` (ai-input), `subfn = [1]` (rag): CoMP-layer commercial classification. The AI System is using the content for retrieval-augmented generation.
- `engineering_intent = [0]` (design-consultation): CLASP-layer workflow classification. The engineer is reading the standard to inform a design decision.
- `target_designation`, `target_edition`: scopes the session to a specific edition of a specific standard.
- `target_clauses` is intentionally omitted: the engineer does not know which clauses are relevant; `search` will surface candidates.
- `deliverable_type = 1` (internal-document): the engineer may paste retrieved text into an internal memo. The Content Owner will not require `exact_text_mode` to be enforced for `synthesized` responses, but exact-text retrieval is available.
- `requester_authority = 0` (individual-licensee): the license is held by the named engineer, not the firm.

---

## Step 2: CoMP handshake response

The Content Owner returns a Package describing the licensed scope, the standard's identity, and the retrieval endpoint.

```json
{
  "package": {
    "id": "PKG-PASC-188-01-2025-S99",
    "title": "PASC-188.01-2025, Hybrid Batch-Continuous Process Control, Part 1",
    "seller": "pasc.example",
    "licenseurl": "https://pasc.example/licensing/clasp-terms",
    "reporturl": "https://pasc.example/reporting/clasp-usage",
    "citation": 1,
    "scope": {
      "scope": 5,
      "ause": 0,
      "pricetype": 3,
      "licensedur": 365,
      "max": 0,
      "ctype": [0],
      "text": [
        {
          "title": "PASC-188.01-2025, Hybrid Batch-Continuous Process Control, Part 1: Models and Terminology",
          "language": [22],
          "pubdate": "2025-03-18T00:00:00Z",
          "author": ["Process Automation Standards Council"],
          "sourcetype": 0
        }
      ],
      "ext": {
        "clasp_version": "0.1",
        "publisher": "pasc.example",
        "designation": "PASC-188.01",
        "edition": "2025",
        "pubdate": "2025-03-18",
        "canonical_language": "en",
        "available_languages": ["en"],
        "supersedes": ["PASC-188.01-2019"],
        "industry_sectors": [0, 1, 2, 8],
        "clause_scheme": "https://pasc.example/schemas/clause-scheme/188-01-2025.json",
        "defined_terms_uri": "https://pasc.example/standards/188-01/2025/defined-terms"
      }
    },
    "retrieval": {
      "auth": 2,
      "endpoint": "https://retrieval.pasc.example/mcp",
      "type": [3],
      "ext": {
        "clasp_version": "0.1",
        "tools": [0, 1, 2, 3, 4],
        "exact_text_mode": 1,
        "citation_envelope": "https://clasp.example/schemas/citation-envelope-0.1.json"
      }
    }
  }
}
```

### What this declares

- `package.citation = 1`: citation is required for any downstream use.
- `scope.ext.industry_sectors`: PASC-188.01 applies to bioprocess, pharmaceutical, chemical, and food-and-beverage process control.
- `scope.ext.clause_scheme`: the URI of the clause-numbering scheme for this edition. Normative for parsing `target_clauses`.
- `retrieval.ext.tools = [0, 1, 2, 3, 4]`: this endpoint exposes `get_clause`, `search`, `resolve_term`, `list_changes`, and `resolve_reference`. It does not expose `compliance_check` (5), which is not relevant for design consultation.
- `retrieval.ext.exact_text_mode = 1`: exact-text retrieval is available even though this session is not stamped.
- `max_artifact_bytes` is not present because `compliance_check` is not offered.

---

## Step 3: MCP tool call — `search`

The AI System opens an MCP connection to the endpoint, authenticates with OAuth2, and calls `search`.

**Tool call:**

```json
{
  "tool": "search",
  "arguments": {
    "query": "phase transition continuous feed",
    "max_results": 5
  }
}
```

**Response:**

```json
{
  "hits": [
    {
      "clause": "6.4.2",
      "snippet": "Phase transitions between a batch unit procedure and a continuous feed state SHALL be modeled as discrete transition states with explicit entry and exit criteria, including material balance reconciliation across the transition boundary.",
      "snippet_offset": 0,
      "snippet_length": 247,
      "clause_total_length": 1842,
      "relevance": 0.91
    },
    {
      "clause": "6.4.3",
      "snippet": "The transition state SHALL persist until the continuous feed flow has reached the steady-state condition defined in the recipe parameter set; see Annex B.2 for example reconciliation calculations.",
      "snippet_offset": 0,
      "snippet_length": 198,
      "clause_total_length": 982,
      "relevance": 0.78
    },
    {
      "clause": "5.2.4",
      "snippet": "Where a unit procedure includes both batch-mode and continuous-mode operations, the recipe SHALL declare the transition trigger as one of: time-based, mass-based, concentration-based, or operator-confirmed.",
      "snippet_offset": 0,
      "snippet_length": 206,
      "clause_total_length": 743,
      "relevance": 0.66
    }
  ],
  "envelope": {
    "envelope_version": "0.1",
    "tool": "search",
    "publisher": "pasc.example",
    "designation": "PASC-188.01",
    "primary_edition": "2025",
    "comparison_edition": null,
    "language": "en",
    "retrieval_timestamp": "2026-05-15T14:23:07Z",
    "endpoint_uri": "https://retrieval.pasc.example/mcp",
    "license_id_fingerprint": "sha256:9a7b5c2e1f3d4a8b6e5c4d3f2a1b0c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b",
    "deliverable_type": 1,
    "requester_authority": 0,
    "citations": [
      {
        "clause": "6.4.2",
        "mode": 2,
        "content_hash": "sha256:1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c",
        "content_length": 247,
        "snippet": { "offset": 0, "length": 247, "of_total_length": 1842 },
        "diff": null,
        "pointer": null
      },
      {
        "clause": "6.4.3",
        "mode": 2,
        "content_hash": "sha256:2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d",
        "content_length": 198,
        "snippet": { "offset": 0, "length": 198, "of_total_length": 982 },
        "diff": null,
        "pointer": null
      },
      {
        "clause": "5.2.4",
        "mode": 2,
        "content_hash": "sha256:3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e",
        "content_length": 206,
        "snippet": { "offset": 0, "length": 206, "of_total_length": 743 },
        "diff": null,
        "pointer": null
      }
    ]
  }
}
```

### What the envelope demonstrates

- One envelope per response; three per-clause `citations` entries because the response references three clauses.
- Every citation has `mode = 2` (snippet). The AI System cannot quote the full clause text from this response, because the response did not contain it; only snippets are present.
- Each `snippet` object carries `offset`, `length`, and `of_total_length`. This lets the AI System tell the user that the snippet is a 247-byte excerpt of an 1842-byte clause, which transparently signals that more clause text exists.
- `content_hash` is computed over the snippet text, not the full clause.

---

## Step 4: MCP tool call — `get_clause`

The AI System decides clause `6.4.2` is the most relevant and retrieves it in full as `exact-text`.

**Tool call:**

```json
{
  "tool": "get_clause",
  "arguments": {
    "clauses": ["6.4.2"],
    "mode": "exact-text"
  }
}
```

**Response:**

```json
{
  "clauses": [
    {
      "clause": "6.4.2",
      "content": "Phase transitions between a batch unit procedure and a continuous feed state SHALL be modeled as discrete transition states with explicit entry and exit criteria, including material balance reconciliation across the transition boundary. The transition state SHALL be identified in the recipe model as a named state, distinct from the preceding batch unit procedure and the succeeding continuous feed state. Entry criteria SHALL include at minimum: confirmation that the preceding unit procedure has reached its declared terminal condition; confirmation that the continuous feed supply system has been verified ready in accordance with 7.3.1. Exit criteria SHALL include at minimum: confirmation that the continuous feed flow has stabilized to within the recipe-declared tolerance; closure of the material balance across the transition window per the reconciliation method declared in 8.2.\n\nNOTE 1: The reconciliation method declared in 8.2 may be a discrete event reconciliation or a continuous integration; the choice is recipe-dependent and is informed by the holdup volume of the transition vessel relative to the continuous feed rate.\n\nNOTE 2: Where the transition involves operator-confirmed entry or exit criteria, the recipe SHALL persist the operator identity, timestamp, and confirmation text in accordance with 9.4.\n\nEXAMPLE: A perfusion-fed fermentation skid transitioning from initial batch growth phase to continuous perfusion feed will commonly use a concentration-based exit criterion (target viable cell density) for the batch phase and a mass-based stability criterion (steady perfusion mass flow over a declared window) for the transition exit. See Annex B.2 for a worked reconciliation example.",
      "mode": "exact-text"
    }
  ],
  "envelope": {
    "envelope_version": "0.1",
    "tool": "get_clause",
    "publisher": "pasc.example",
    "designation": "PASC-188.01",
    "primary_edition": "2025",
    "comparison_edition": null,
    "language": "en",
    "retrieval_timestamp": "2026-05-15T14:23:24Z",
    "endpoint_uri": "https://retrieval.pasc.example/mcp",
    "license_id_fingerprint": "sha256:9a7b5c2e1f3d4a8b6e5c4d3f2a1b0c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b",
    "deliverable_type": 1,
    "requester_authority": 0,
    "citations": [
      {
        "clause": "6.4.2",
        "mode": 0,
        "content_hash": "sha256:4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f",
        "content_length": 1842,
        "snippet": null,
        "diff": null,
        "pointer": null
      }
    ]
  }
}
```

### What this demonstrates

- A second envelope, this time for `get_clause`. The `license_id_fingerprint` matches Step 3's envelope: both responses are within the same session.
- `citations[0].mode = 0` (exact-text): the AI System may now quote `6.4.2` verbatim downstream, accompanied by this envelope.
- `content_length = 1842` matches the `of_total_length` reported in the search snippet. The AI System can verify it has received the whole clause.
- `content_hash` is over the full clause text and differs from the search-response hash for the same clause (which was over a 247-byte snippet of it). This is intentional: the hash is over what was actually returned, not over a canonical clause identity.

---

## What this example demonstrates that the others do not

- The `search` to `get_clause` chain pattern.
- Two distinct response modes (snippet, exact-text) inside one session, each producing a separate envelope.
- The fact that the engineer's natural-language query is an MCP tool argument and does not appear in the CoMP handshake.
- A session where `target_clauses` is intentionally omitted at the handshake layer because the engineer does not know what is relevant.
- The lowest-strictness deliverable type, `internal-document`, where `exact_text_mode` is available but not enforced.
- An `individual-licensee` session.

## Notes on the protocol surface

- `aisysuse.subfn` is set to `[1]` (rag) per CoMP. This is consistent with `engineering_intent = [0]` (design-consultation). The two fields are unioned, not redundant.
- `aisysuse.scope` is set to `5` (curated selection per CoMP), which is the closest fit for "scoped to a specific edition of a specific standard". The CLASP `target_designation` and `target_edition` in `ext` carry the actual specificity.
- The `endpoint_uri` field of the envelope and the `uri` field of `aisystemuse` are deliberately the same MCP endpoint in this example; production deployments may differ if the handshake and tool calls go through different proxies.
