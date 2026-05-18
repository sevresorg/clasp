# Example 02: Cross-edition comparison supporting a regulatory submission

**Scenario:** a process engineer at a small-molecule pharmaceutical sponsor is preparing a CMC supplement for a regulatory submission. The submission switches a tablet line from batch to continuous direct compression. The relevant verification-of-continuous-manufacturing standard has just gone through a major edition revision.

**Tools exercised:** `list_changes`, `resolve_reference` (the second call surfaces the cross-publisher pointer case).

**Standard (fictional):** `TMF-3050, Standard Guide for Verification of Continuous Pharmaceutical Manufacturing Systems`. Comparison between editions `2023` and `2026`.

**Engineering intent:** `regulatory-submission` (5).

**Deliverable type:** `submitted-to-regulator` (4).

**Requester authority:** `organization-licensee` (1). Regulatory submissions are issued by the firm, not by the individual engineer.

**CLASP version:** `0.1`.

**Language:** canonical `en`; AI System declares `preferred_languages: ["en"]`. The regulatory incorporation is annotated with the transition-window fields and the BPEC pointer in Step 5 closes the loop with the [BPEC discoverability catalog example](./bpec-catalog.md).

---

## Engineering context

The sponsor's existing validation procedures were qualified against `TMF-3050-2023`, and the regulator's reviewer guidance, last updated in 2024, references the 2023 edition by name. `TMF-3050-2026` is now in force and the sponsor needs a structured, citable diff between the two editions to support the supplement's justification narrative and to identify which qualified procedures need re-evaluation.

The engineer's regulatory-authoring environment hosts a CLASP-aware AI assistant. The AI System issues `list_changes` between the two editions, then follows a normative reference in the 2026 edition via `resolve_reference`. The reference points to a clause in `BPEC-200, Hygienic Design of Single-Use Bioprocess Components`, which is published by a different SDO that the sponsor does not hold a license to. The response is a structured pointer, not text.

---

## Step 1: CoMP handshake request

```json
{
  "aisystem": {
    "name": "reg-authoring.example",
    "ua": "RegSubmissionAI/2.0",
    "id": "TL-REG-998877",
    "aisysuse": {
      "lid": "TMF-LIC-SPONSOR-2026-A7714",
      "aiauth": 2,
      "uri": ["https://retrieval.tmf.example/mcp"],
      "scope": 5,
      "function": [3],
      "subfn": [2],
      "resdis": 1,
      "ext": {
        "clasp_version": "0.1",
        "engineering_intent": [5],
        "target_designation": ["TMF-3050"],
        "target_edition": "2023",
        "target_edition_compare": "2026",
        "preferred_languages": ["en"],
        "deliverable_type": 4,
        "requester_authority": 1
      }
    }
  }
}
```

### What this declares

- `function = [3]` (ai-input), `subfn = [2]` (grounding): the AI System will use retrieved content to ground generated text in the regulatory submission.
- `engineering_intent = [5]` (regulatory-submission): the engineering workflow is preparing a regulator-bound document.
- `target_edition = "2023"`, `target_edition_compare = "2026"`: the session intends to use `list_changes` between these two editions. With `target_edition_compare` populated, `target_edition` is interpreted as the "from" edition.
- `deliverable_type = 4` (submitted-to-regulator): the strictest deliverable type. The Content Owner is expected to require `exact_text_mode = 1` and may impose other constraints (some publishers require `requester_authority = 0` for submitted-to-regulator; this publisher accepts organization-licensee).
- `requester_authority = 1` (organization-licensee): the firm is the licensee. The citation envelope will carry this through and any FDA reviewer can verify the firm-level attribution.

---

## Step 2: CoMP handshake response

```json
{
  "package": {
    "id": "PKG-TMF-3050-CROSSED-S122",
    "title": "TMF-3050 Cross-Edition Access (2023 to 2026)",
    "seller": "tmf.example",
    "licenseurl": "https://tmf.example/licensing/clasp-terms",
    "reporturl": "https://tmf.example/reporting/clasp-usage",
    "citation": 1,
    "scope": {
      "scope": 5,
      "ause": 0,
      "pricetype": 4,
      "pricetier": 2,
      "licensedur": 365,
      "max": 0,
      "ctype": [0],
      "text": [
        {
          "title": "TMF-3050-2023, Standard Guide for Verification of Continuous Pharmaceutical Manufacturing Systems",
          "language": [22],
          "pubdate": "2023-09-01T00:00:00Z",
          "author": ["Technical Methods Federation"],
          "sourcetype": 0
        },
        {
          "title": "TMF-3050-2026, Standard Guide for Verification of Continuous Pharmaceutical Manufacturing Systems",
          "language": [22],
          "pubdate": "2026-02-12T00:00:00Z",
          "author": ["Technical Methods Federation"],
          "sourcetype": 0
        }
      ],
      "ext": {
        "clasp_version": "0.1",
        "publisher": "tmf.example",
        "designation": "TMF-3050",
        "edition": "2026",
        "pubdate": "2026-02-12",
        "canonical_language": "en",
        "available_languages": ["en", "fr"],
        "supersedes": ["TMF-3050-2023"],
        "industry_sectors": [0, 1],
        "clause_scheme": "https://tmf.example/schemas/clause-scheme/3050-2026.json",
        "defined_terms_uri": "https://tmf.example/standards/3050/2026/defined-terms",
        "normative_refs": [
          {
            "designation": "BPEC-200",
            "edition": "2024",
            "publisher": "bpec.example",
            "clauses": ["8.3.2"]
          },
          {
            "designation": "TMF-3001",
            "edition": "2022",
            "publisher": "tmf.example",
            "clauses": ["4.1", "4.2"]
          }
        ],
        "regulatory_incorporations": [
          {
            "regulator": "example-pharma-regulator.gov",
            "regulation_id": "EX-CFR-211.X (proposed amendment, 2024)",
            "incorporation_date": "2024-11-01",
            "pinned_edition": "2023",
            "transition_window_start": "2026-07-01",
            "transition_window_end": "2027-12-31",
            "next_pinned_edition": "2026"
          }
        ]
      }
    },
    "retrieval": {
      "auth": 2,
      "endpoint": "https://retrieval.tmf.example/mcp",
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

- `scope.ext.edition` is the "current" edition (`2026`). The `supersedes` chain records that `2023` is the prior edition. The `target_edition` from the handshake (`2023`) is licensed for this session even though it is superseded; cross-edition tools require both editions to be accessible.
- `regulatory_incorporations` is populated: the regulator pinned the 2023 edition. The transition-window fields (`transition_window_start: 2026-07-01`, `transition_window_end: 2027-12-31`, `next_pinned_edition: 2026`) tell the AI System and the engineer that the regulator has announced a transition. The 2026 edition becomes acceptable for new submissions on 2026-07-01, and the 2023 edition is no longer acceptable after 2027-12-31. This is the architectural reason the engineer is generating a diff in the first place; the submission must justify why work qualified against the pinned edition is acceptable under the current edition, and the transition-window dates give the regulator's reviewer a concrete frame for the justification.
- `normative_refs` lists two normative references. One is to a TMF document (same publisher; the responding Content Owner can resolve it inline). The other is to `BPEC-200`, a different publisher; resolution will require a pointer.
- `retrieval.ext.tools = [0, 1, 2, 3, 4]` includes `list_changes` (3) and `resolve_reference` (4), which is what this session needs.

---

## Step 3: MCP tool call — `list_changes`

The AI System requests the change set between the two editions, filtered to the specific clause subtree that is operative for the supplement (the verification of continuous direct compression sub-process is covered in clauses 7.x).

**Tool call:**

```json
{
  "tool": "list_changes",
  "arguments": {
    "clause_filter": "7..7.99"
  }
}
```

**Response (abbreviated to three changes for readability; a real response would include all changes in the filter):**

```json
{
  "changes": [
    {
      "clause": "7.2.1",
      "change_kind": "modified",
      "before_text": "The verification plan SHALL include sampling at the outlet of the continuous direct compression line at a frequency sufficient to detect material composition drift over a single production campaign.",
      "after_text": "The verification plan SHALL include sampling at the outlet of the continuous direct compression line at a frequency sufficient to detect material composition drift over a single production campaign. Sampling frequency SHALL be defined as a function of the line's mean residence time, with a minimum of three samples per residence time interval, in accordance with the residence-time-distribution analysis required by 7.2.2.",
      "description": "Sampling frequency now has a normative minimum tied to mean residence time. Prior text was silent on minimum frequency. Impact: validation procedures qualifying sampling at fixed wall-clock intervals (e.g. every 15 minutes) must be reassessed against the new mean-residence-time-coupled criterion."
    },
    {
      "clause": "7.2.2",
      "change_kind": "added",
      "before_text": null,
      "after_text": "A residence-time-distribution analysis SHALL be performed for the continuous direct compression line as part of the verification plan. The analysis SHALL use the method described in TMF-3001:2022 clauses 4.1 and 4.2, or an equivalent method documented in the verification plan and accepted by the responsible quality function.",
      "description": "New normative clause introducing residence-time-distribution analysis as a verification-plan requirement. Cross-references TMF-3001:2022."
    },
    {
      "clause": "7.4.3",
      "change_kind": "modified",
      "before_text": "Wetted components in contact with active pharmaceutical ingredient streams SHALL comply with the hygienic design requirements of generally recognized industry practice.",
      "after_text": "Wetted components in contact with active pharmaceutical ingredient streams SHALL comply with BPEC-200:2024 clause 8.3.2.",
      "description": "Replaces 'generally recognized industry practice' with a normative reference to BPEC-200:2024 clause 8.3.2. Impact: components qualified under the prior text need a documented BPEC-200:2024 compliance assessment."
    }
  ],
  "envelope": {
    "envelope_version": "0.1",
    "tool": "list_changes",
    "publisher": "tmf.example",
    "designation": "TMF-3050",
    "primary_edition": "2023",
    "comparison_edition": "2026",
    "language": "en",
    "retrieval_timestamp": "2026-05-15T15:08:11Z",
    "endpoint_uri": "https://retrieval.tmf.example/mcp",
    "license_id_fingerprint": "sha256:6f5e4d3c2b1a0f9e8d7c6b5a4938271605f4e3d2c1b0a9988776655443322110",
    "deliverable_type": 4,
    "requester_authority": 1,
    "citations": [
      {
        "clause": "7.2.1",
        "mode": 3,
        "content_hash": "sha256:8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b",
        "content_length": 692,
        "snippet": null,
        "diff": { "from_edition": "2023", "to_edition": "2026", "change_kind": "modified" },
        "pointer": null
      },
      {
        "clause": "7.2.2",
        "mode": 3,
        "content_hash": "sha256:9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c",
        "content_length": 401,
        "snippet": null,
        "diff": { "from_edition": "2023", "to_edition": "2026", "change_kind": "added" },
        "pointer": null
      },
      {
        "clause": "7.4.3",
        "mode": 3,
        "content_hash": "sha256:0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d",
        "content_length": 328,
        "snippet": null,
        "diff": { "from_edition": "2023", "to_edition": "2026", "change_kind": "modified" },
        "pointer": null
      }
    ]
  }
}
```

### What this demonstrates

- The envelope's `primary_edition` is `2023` (the "from" edition) and `comparison_edition` is `2026`. This is the one tool that populates both fields.
- All three citations have `mode = 3` (diff). Each `diff` sub-object records the editions compared and the kind of change. An AI System emitting a claim that "clause 7.2.2 is new in the 2026 edition" can be checked against a `change_kind = added` diff citation; no other mode supports such a claim.
- The `content_hash` for each citation is computed over the canonical JSON serialization of the diff record, not over the clause text. The change_kind and the before/after text are part of the hashed structure, so a downstream reader can recompute the hash to verify integrity.

---

## Step 4: MCP tool call — `resolve_reference` (same-publisher)

The 2026 edition's clause `7.2.2` normatively cites `TMF-3001:2022` clauses `4.1` and `4.2`. The AI System resolves this reference. Because both designations belong to the same publisher (TMF), the response returns the clauses inline.

**Tool call:**

```json
{
  "tool": "resolve_reference",
  "arguments": {
    "reference": {
      "designation": "TMF-3001",
      "edition": "2022",
      "clauses": ["4.1", "4.2"]
    },
    "mode": "exact-text"
  }
}
```

**Response (showing just clause 4.1; clause 4.2 would follow the same pattern):**

```json
{
  "resolved": {
    "designation": "TMF-3001",
    "edition": "2022",
    "publisher": "tmf.example",
    "clauses": [
      {
        "clause": "4.1",
        "content": "The residence-time-distribution (RTD) analysis of a continuous process line SHALL be performed by introducing a tracer at the line inlet and measuring tracer concentration at the line outlet over time. The tracer SHALL be inert with respect to the process chemistry and detectable at the required sensitivity at the outlet measurement point. The method of tracer introduction SHALL be either pulse, step, or random binary sequence; the choice SHALL be documented in the analysis plan.",
        "mode": "exact-text"
      }
    ]
  },
  "envelope": {
    "envelope_version": "0.1",
    "tool": "resolve_reference",
    "publisher": "tmf.example",
    "designation": "TMF-3001",
    "primary_edition": "2022",
    "comparison_edition": null,
    "language": "en",
    "retrieval_timestamp": "2026-05-15T15:09:42Z",
    "endpoint_uri": "https://retrieval.tmf.example/mcp",
    "license_id_fingerprint": "sha256:6f5e4d3c2b1a0f9e8d7c6b5a4938271605f4e3d2c1b0a9988776655443322110",
    "deliverable_type": 4,
    "requester_authority": 1,
    "citations": [
      {
        "clause": "4.1",
        "mode": 0,
        "content_hash": "sha256:1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e",
        "content_length": 487,
        "snippet": null,
        "diff": null,
        "pointer": null
      }
    ]
  }
}
```

### What this demonstrates

- The envelope's `designation` is `TMF-3001`, not `TMF-3050`. A single CLASP session can return citation envelopes anchored to different standards as long as all such standards are within the session's licensed scope.
- `mode = 0` (exact-text): the cited text is verbatim. The AI System may quote it in the regulatory submission as long as the envelope is attached.

---

## Step 5: MCP tool call — `resolve_reference` (cross-publisher)

The 2026 edition's clause `7.4.3` normatively cites `BPEC-200:2024` clause `8.3.2`. BPEC is a different publisher. The sponsor does not hold a BPEC license.

**Tool call:**

```json
{
  "tool": "resolve_reference",
  "arguments": {
    "reference": {
      "designation": "BPEC-200",
      "edition": "2024",
      "clauses": ["8.3.2"]
    },
    "mode": "exact-text"
  }
}
```

**Response:**

```json
{
  "resolved": {
    "designation": "BPEC-200",
    "edition": "2024",
    "publisher": "bpec.example",
    "clauses": [
      {
        "clause": "8.3.2",
        "pointer": {
          "target_publisher": "bpec.example",
          "target_designation": "BPEC-200",
          "target_edition": "2024",
          "target_clause": "8.3.2",
          "discoverability_uri": "https://bpec.example/.well-known/clasp-catalog"
        }
      }
    ]
  },
  "envelope": {
    "envelope_version": "0.1",
    "tool": "resolve_reference",
    "publisher": "tmf.example",
    "designation": "BPEC-200",
    "primary_edition": "2024",
    "comparison_edition": null,
    "language": null,
    "retrieval_timestamp": "2026-05-15T15:10:18Z",
    "endpoint_uri": "https://retrieval.tmf.example/mcp",
    "license_id_fingerprint": "sha256:6f5e4d3c2b1a0f9e8d7c6b5a4938271605f4e3d2c1b0a9988776655443322110",
    "deliverable_type": 4,
    "requester_authority": 1,
    "citations": [
      {
        "clause": "8.3.2",
        "mode": 5,
        "content_hash": "sha256:2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f",
        "content_length": 218,
        "snippet": null,
        "diff": null,
        "pointer": {
          "target_publisher": "bpec.example",
          "target_designation": "BPEC-200",
          "target_edition": "2024",
          "target_clause": "8.3.2",
          "discoverability_uri": "https://bpec.example/.well-known/clasp-catalog"
        }
      }
    ]
  }
}
```

### What this demonstrates

- The envelope's `publisher` (`tmf.example`) and the citation's `pointer.target_publisher` (`bpec.example`) are different. This is the cross-publisher case.
- `mode = 5` (pointer): the AI System has NOT received the text of `BPEC-200:2024` clause `8.3.2`. It has received a structured pointer to where that text can be licensed. Downstream emission rules apply: the AI System MUST NOT quote `BPEC-200` clause `8.3.2` based on this envelope. It can disclose to the engineer that the reference exists and direct them to the BPEC discoverability catalog.
- `content_hash` is over the canonical JSON serialization of the `pointer` object, not over clause text. There is no clause text to hash.
- `endpoint_uri` is the TMF endpoint, because the TMF endpoint generated the pointer; the AI System would need a separate handshake with BPEC to retrieve the actual clause text.
- The envelope's `language` field is `null`. Pointer-only responses do not include any rendered text, so no language attaches; this is the only envelope shape that legitimately has a `null` language.

---

## What this example demonstrates that the others do not

- `target_edition_compare` populated; the envelope's `primary_edition` and `comparison_edition` both populated for `list_changes`.
- `mode = 3` (diff) and `mode = 5` (pointer) citation types.
- `submitted-to-regulator` deliverable type forcing `exact_text_mode = 1` and the strictest envelope discipline.
- `organization-licensee` `requester_authority`.
- A single session spanning citations from two designations (`TMF-3050`, `TMF-3001`) and a pointer to a third (`BPEC-200`).
- `regulatory_incorporations` populated with transition-window fields (`transition_window_start`, `transition_window_end`, `next_pinned_edition`), capturing why the engineer is doing the diff and giving the regulator's reviewer a concrete frame for the transition justification.
- `available_languages` declaring more than one rendering (`["en", "fr"]`); the AI System's `preferred_languages: ["en"]` selects English, but a French-language retrieval would have been available within the same session.
- A `language: null` envelope (Step 5), demonstrating the cross-publisher pointer case where no language attaches.

## Notes on the protocol surface

- The `clause_filter` argument to `list_changes` is an MCP tool argument and is not part of the CoMP handshake. It is also not part of `target_clauses` in the handshake; `target_clauses` declares the session's authorized clause subset (here, none specified, meaning the full standard is in scope), while `clause_filter` narrows the specific tool call.
- The `target_clauses` renumbering case is handled by CLASP-0.1: default is the "to" edition's numbering, with explicit override via `target_clauses_numbering`. This example continues to leave `target_clauses` unset at session level (filtering at tool-call time) because the scenario doesn't need to narrow the diff; the mechanism is available for scenarios that do.
- The `Object: RegulatoryIncorporation` shape used here is the CLASP-0.1 form, including the transition-window fields. The multi-jurisdiction case is addressed in CLASP-0.1 prose: the Content Owner surfaces all incorporations; the AI System filters by session context. This example shows only one regulator; a multi-regulator example would just populate the array with multiple entries and let the AI System pick by deliverable target.
