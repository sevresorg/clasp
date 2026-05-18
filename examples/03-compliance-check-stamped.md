# Example 03: Cause-and-effect matrix gap analysis for a stamped deliverable

**Scenario:** a process safety engineer at an engineering, procurement, and construction (EPC) firm is reviewing a draft cause-and-effect (C&E) matrix for the safety instrumented system (SIS) of a hydroprocessing unit before applying their Professional Engineer seal.

**Tools exercised:** `compliance_check`, `resolve_term`.

**Standard (fictional):** `PASC-184.02-2025, Cause-and-Effect Matrix Conformance for Safety Instrumented Systems`.

**Engineering intent:** `gap-analysis` (1).

**Deliverable type:** `stamped-deliverable` (3).

**Requester authority:** `individual-licensee` (0). The PE seal attaches liability to the named engineer.

**CLASP version:** `0.1`.

**Language:** canonical `en`; AI System declares `preferred_languages: ["en"]`. Stamped deliverables require canonical-language attestation; see envelope obligations in the spec.

---

## Engineering context

The C&E matrix was authored partly by hand and partly by an upstream tool. Before applying their PE seal, the engineer wants an independent gap analysis against the specific conformance clauses of the standard (clauses 7.2, 7.3, and Annex B). The engineer's workstation runs a CLASP-aware AI agent that already holds a license to the standard.

The AI agent submits the draft C&E matrix as a CSV file to `compliance_check` and receives a structured list of findings, each anchored to one or more clauses. The agent then issues a `resolve_term` for the defined term "safe state" because one finding hinges on the engineer's interpretation of that term matching the standard's vocabulary clause.

---

## Step 1: CoMP handshake request

```json
{
  "aisystem": {
    "name": "epc-safety-review.example",
    "ua": "SafetyReviewAI/3.2",
    "id": "TL-REG-445566",
    "aisysuse": {
      "lid": "PASC-LIC-INDIV-PE-2026-71140",
      "aiauth": 2,
      "uri": ["https://retrieval.pasc.example/mcp"],
      "scope": 5,
      "function": [1],
      "subfn": [4],
      "resdis": 1,
      "ext": {
        "clasp_version": "0.1",
        "engineering_intent": [1],
        "target_designation": ["PASC-184.02"],
        "target_edition": "2025",
        "target_clauses": ["7.2", "7.3", "Annex B"],
        "preferred_languages": ["en"],
        "requester_sectors": [3, 2],
        "deliverable_type": 3,
        "requester_authority": 0
      }
    }
  }
}
```

### What this declares

- `function = [1]` (ai-all): the agent performs analysis that goes beyond pure retrieval; CoMP's `ai-all` is the broadest commercial classification.
- `subfn = [4]` (agent-actions): the agent is taking an action on behalf of the engineer (running a check), not just consuming content.
- `engineering_intent = [1]` (gap-analysis): the engineering workflow is comparing an artifact against a standard.
- `target_clauses = ["7.2", "7.3", "Annex B"]`: the session is narrowly scoped to the conformance clauses and the relevant annex. `Annex B` is a top-level annex reference; clause-level addressing inside the annex would use `Annex B.1`, `Annex B.2`, etc.
- `deliverable_type = 3` (stamped-deliverable): the engineer will apply a PE seal. The Content Owner is expected to require `exact_text_mode = 1`.
- `requester_authority = 0` (individual-licensee): the license is held by the PE personally; liability for the stamped deliverable attaches to that individual.
- `requester_sectors = [3, 2]` (oil-and-gas, chemical): the PE's EPC firm operates in both sectors and the hydroprocessing project sits at their intersection. The Content Owner may use this to apply sector-conditional rules in `compliance_check` (clauses that apply differently to refinery service versus chemical-process service).

---

## Step 2: CoMP handshake response

```json
{
  "package": {
    "id": "PKG-PASC-184-02-2025-S77",
    "title": "PASC-184.02-2025, Cause-and-Effect Matrix Conformance for SIS",
    "seller": "pasc.example",
    "licenseurl": "https://pasc.example/licensing/clasp-terms-pe",
    "reporturl": "https://pasc.example/reporting/clasp-usage",
    "citation": 1,
    "scope": {
      "scope": 5,
      "ause": 0,
      "pricetype": 1,
      "licensedur": 365,
      "max": 1,
      "ctype": [0],
      "text": [
        {
          "title": "PASC-184.02-2025, Cause-and-Effect Matrix Conformance for SIS",
          "language": [22],
          "pubdate": "2025-06-04T00:00:00Z",
          "author": ["Process Automation Standards Council"],
          "sourcetype": 0
        }
      ],
      "ext": {
        "clasp_version": "0.1",
        "publisher": "pasc.example",
        "designation": "PASC-184.02",
        "edition": "2025",
        "pubdate": "2025-06-04",
        "canonical_language": "en",
        "available_languages": ["en"],
        "supersedes": ["PASC-184.02-2018"],
        "industry_sectors": [2, 3, 4],
        "clause_scheme": "https://pasc.example/schemas/clause-scheme/184-02-2025.json",
        "defined_terms_uri": "https://pasc.example/standards/184-02/2025/defined-terms",
        "normative_refs": [
          {
            "designation": "PASC-184.01",
            "edition": "2022",
            "publisher": "pasc.example",
            "clauses": ["5.3", "6.1"]
          }
        ]
      }
    },
    "retrieval": {
      "auth": 2,
      "endpoint": "https://retrieval.pasc.example/mcp",
      "type": [3],
      "ext": {
        "clasp_version": "0.1",
        "tools": [0, 1, 2, 4, 5],
        "exact_text_mode": 1,
        "citation_envelope": "https://clasp.example/schemas/citation-envelope-0.1.json",
        "max_artifact_bytes": 4194304
      }
    }
  }
}
```

### What this declares

- `industry_sectors = [2, 3, 4]` (chemical, oil-and-gas, power-generation): the standard applies across multiple process-industry sectors. The handshake's `requester_sectors = [3, 2]` (oil-and-gas, chemical) declares which sectors the requester operates in; the intersection (`{2, 3}`) is the operative scope for sector-conditional rules. Power-generation clauses in this standard are not relevant to this session. The ISIC Rev 4 cross-reference for `[2, 3]` is `C20` and `B06`, available to downstream integrators via the [List: Industry Sector](../CLASP-0.1.md#list-industry-sector) table.
- `retrieval.ext.tools = [0, 1, 2, 4, 5]`: this endpoint exposes `get_clause`, `search`, `resolve_term`, `resolve_reference`, and `compliance_check`. It does not expose `list_changes` for this license tier.
- `retrieval.ext.exact_text_mode = 1` is required by the `stamped-deliverable` declaration. The Content Owner has authorized the session under this constraint.
- `retrieval.ext.max_artifact_bytes = 4194304` (4 MiB): the artifact submitted to `compliance_check` MUST NOT exceed this size.
- `scope.max = 1`: per CoMP, there is an upper limit on how much can be consumed in this session; details are in the license terms.

---

## Step 3: MCP tool call — `compliance_check`

The AI agent submits the draft C&E matrix as a CSV payload (here abbreviated to a single representative row for readability) against the declared clause set.

**Tool call (artifact abbreviated):**

```json
{
  "tool": "compliance_check",
  "arguments": {
    "artifact": {
      "content_type": "text/csv",
      "encoding": "utf-8",
      "bytes": "Q2F1c2VfSUQsQ2F1c2VfRGVzY3JpcHRpb24sRWZmZWN0X0lELEVmZmVjdF9EZXNjcmlwdGlvbixUaW1lX3RvX1NhZmVfU3RhdGUKQy0wMDEsRmVlZF9ndWFyZF9oaWdoX3ByZXNzdXJlX3RyaXAsRS0wMTAsSXNvbGF0ZV9mZWVkX3ZhbHZlX1hWLTEwMSwxMHMKQy0wMDIsUmVhY3Rvcl9oaWdoX3RlbXBlcmF0dXJlX3RyaXAsRS0wMjEsRmFpbF9zYWZlX29wZW5fcXVlbmNoX3ZhbHZlX1hWLTIyMyw1cw=="
    },
    "clauses": ["7.2", "7.3", "Annex B.1", "Annex B.2"]
  }
}
```

(The `bytes` field is base64-encoded; decoded it begins with a CSV header `Cause_ID,Cause_Description,Effect_ID,Effect_Description,Time_to_Safe_State` and contains the relevant rows.)

**Response:**

```json
{
  "findings": [
    {
      "finding_id": "F-001",
      "severity": 0,
      "referenced_clauses": ["7.2"],
      "artifact_location": "row 2, columns Cause_ID through Effect_Description",
      "explanation": "Row 2 declares a single cause (C-001) mapping to a single effect (E-010) with both identifiers and descriptions populated. This satisfies clause 7.2 requirements for unambiguous cause-to-effect identification."
    },
    {
      "finding_id": "F-002",
      "severity": 2,
      "referenced_clauses": ["7.3", "Annex B.1"],
      "artifact_location": "row 3, column Time_to_Safe_State",
      "explanation": "Row 3 declares Time_to_Safe_State as 5 seconds for effect E-021 (fail-safe open quench valve XV-223). Clause 7.3 requires that the time-to-safe-state value be supported by either a calculation reference or a test reference; the matrix row contains no such reference and the accompanying documentation submitted does not include one. Annex B.1 specifies the documentation expected to support sub-10-second declarations specifically. This is a gap relative to both the body clause and the annex."
    },
    {
      "finding_id": "F-003",
      "severity": 1,
      "referenced_clauses": ["Annex B.2"],
      "artifact_location": "row 3, column Effect_Description",
      "explanation": "Effect E-021 is described as 'fail-safe open quench valve XV-223'. Annex B.2 advises that fail-safe valve effects be described in terms of the resulting process state (e.g., 'quench flow established') rather than the valve action ('valve opened'), so that the matrix is auditable against the safe-state definition independent of valve implementation. This is advisory, not a gap."
    }
  ],
  "envelope": {
    "envelope_version": "0.1",
    "tool": "compliance_check",
    "publisher": "pasc.example",
    "designation": "PASC-184.02",
    "primary_edition": "2025",
    "comparison_edition": null,
    "language": "en",
    "retrieval_timestamp": "2026-05-15T16:42:55Z",
    "endpoint_uri": "https://retrieval.pasc.example/mcp",
    "license_id_fingerprint": "sha256:3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5e4f3a2b",
    "deliverable_type": 3,
    "requester_authority": 0,
    "citations": [
      {
        "clause": "7.2",
        "mode": 4,
        "content_hash": "sha256:5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b",
        "content_length": 412,
        "snippet": null,
        "diff": null,
        "pointer": null
      },
      {
        "clause": "7.3",
        "mode": 4,
        "content_hash": "sha256:6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c",
        "content_length": 736,
        "snippet": null,
        "diff": null,
        "pointer": null
      },
      {
        "clause": "Annex B.1",
        "mode": 4,
        "content_hash": "sha256:7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d",
        "content_length": 736,
        "snippet": null,
        "diff": null,
        "pointer": null
      },
      {
        "clause": "Annex B.2",
        "mode": 4,
        "content_hash": "sha256:8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e",
        "content_length": 324,
        "snippet": null,
        "diff": null,
        "pointer": null
      }
    ]
  }
}
```

### What this demonstrates

- One envelope per response, with one citation per **unique** referenced clause. Finding `F-002` references two clauses (`7.3` and `Annex B.1`); both appear in the citations array exactly once each (de-duplicated across findings).
- Every citation has `mode = 4` (structured-analysis). The `content_hash` for each citation is computed over the canonical JSON serialization of the finding object(s) referencing that clause, sorted by `finding_id`. So the hash for clause `7.3` is over `F-002`; the hash for clause `Annex B.1` is over `F-002` as well (same finding); they happen to be equal in `content_length` because both reference the same finding.
- The artifact content is NOT in the envelope. The Content Owner does not retain the artifact (per the protocol's compliance_check retention rule) and the citation envelope is over the **findings** (the Content Owner's output), not the artifact (the AI System's input).
- An AI System emitting a gap-analysis report to the engineer must attach this envelope. The report can paraphrase findings, but if it quotes any clause text from `PASC-184.02`, it must do so through a separate `get_clause` envelope; `mode = 4` is not sufficient authority for verbatim clause text.
- `artifact_location` uses CSV row/column addressing because the artifact's content type is `text/csv`. For a different content type the format would differ (PDF page/paragraph, code file line, etc).

---

## Step 4: MCP tool call — `resolve_term`

The engineer flags `F-003` for review and the agent retrieves the standard's definition of "safe state" so the engineer can confirm the matrix's terminology is consistent.

**Tool call:**

```json
{
  "tool": "resolve_term",
  "arguments": {
    "term": "safe state"
  }
}
```

**Response:**

```json
{
  "definitions": [
    {
      "clause": "3.1.17",
      "term": "safe state",
      "definition": "state of the equipment under control (EUC) in which an identified hazard cannot be realized, or in which the consequences of the realized hazard remain within tolerable limits as specified in the safety requirements specification. NOTE 1: A safe state may require continued operation of one or more safety functions. NOTE 2: For a continuous process, the safe state is normally a continued operating state distinct from full shutdown. NOTE 3: For a batch process, the safe state may be defined per-phase.",
      "mode": "exact-text"
    }
  ],
  "envelope": {
    "envelope_version": "0.1",
    "tool": "resolve_term",
    "publisher": "pasc.example",
    "designation": "PASC-184.02",
    "primary_edition": "2025",
    "comparison_edition": null,
    "language": "en",
    "retrieval_timestamp": "2026-05-15T16:43:14Z",
    "endpoint_uri": "https://retrieval.pasc.example/mcp",
    "license_id_fingerprint": "sha256:3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5e4f3a2b",
    "deliverable_type": 3,
    "requester_authority": 0,
    "citations": [
      {
        "clause": "3.1.17",
        "mode": 0,
        "content_hash": "sha256:9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f",
        "content_length": 558,
        "snippet": null,
        "diff": null,
        "pointer": null
      }
    ]
  }
}
```

### What this demonstrates

- `clause 3.1.17` is OUTSIDE the session's declared `target_clauses` (`7.2`, `7.3`, `Annex B`). The Content Owner authorized `resolve_term` access to the standard's vocabulary clauses (3.x in this standard) even though they were not enumerated in `target_clauses`, because the vocabulary is supporting infrastructure for the declared clauses. Whether this is universally OK or whether a Content Owner can decline such resolutions is something the spec leaves to the Content Owner's policy; the CLASP-0.1 spec does not normatively require either behavior.
- `mode = 0` (exact-text): the definition is verbatim. The engineer can quote it in the stamped deliverable's defined-terms section accompanied by this envelope.

---

## What this example demonstrates that the others do not

- `compliance_check`: the most architecturally novel CLASP tool, with an artifact input and a structured findings output.
- `mode = 4` (structured-analysis) citations and how they hash over findings rather than over clause text.
- `resolve_term` and how it can resolve clauses outside the declared `target_clauses` set.
- `stamped-deliverable` deliverable type with `requester_authority = individual-licensee` (the PE-specific liability case).
- CoMP `subfn = [4]` (agent-actions): the AI is performing an action, not just retrieving.
- `target_clauses` containing both numbered clauses and an annex-level reference (`Annex B`).
- `requester_sectors` populated, demonstrating the field that lets AI Systems declare the requester's sector on the handshake.
- A CoMP `max = 1` package, indicating that license terms cap the volume of use.

## Notes on the protocol surface

- The annex addressing used here is `Annex B` at the session level (the broadest scope) and `Annex B.1`, `Annex B.2` at the tool-call level (specific sub-clauses). The `clause_scheme` URI is the authority for this syntax.
- The artifact is base64-encoded inside the MCP tool call. Norma will specify whether large artifacts should use a side-channel upload instead. For 4 MiB this inline encoding is acceptable.
- The Content Owner's policy on artifact retention is communicated through the license terms, not through the protocol. The protocol says the Content Owner MUST NOT retain the artifact beyond the tool call's duration unless the license explicitly permits.
- The `requester_authority = 0` declaration on a `stamped-deliverable` is the case where the citation envelope's downstream attribution is to a named individual. The `license_id_fingerprint` is a one-way hash of the license ID, so a downstream reviewer (a regulator, a client) can verify that the envelope's fingerprint matches the engineer's license without the engineer's license ID being exposed in the deliverable.
