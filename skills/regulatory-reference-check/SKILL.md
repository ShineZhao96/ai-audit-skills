---
name: regulatory-reference-check
description: "Standardized Reference Document Verification — inspects corporate policy/standard documents for references (laws, regulations, standards, parent-company policies, in-house policies) that are missing, misnamed, misnumbered, abolished, or superseded. Triggers: 核查规范性引用文件 / 引用文件版本检查 / 法规有效性核查 / 引用文件校验 / 制度合规审查 / 引用文件清理."
---

# regulatory-reference-check

> Standardized Reference Document Verification Skill

---

## Description

Inspects a corporate policy / standard / regulation document's reference list
and the body text, and detects candidate issues in five disjoint categories:

| # | Issue | Meaning |
|---|-------|---------|
| 1 | `missing-reference` | Body mentions a regulation / standard / policy but the reference list omits it. |
| 2 | `name-change` | Cited name does not match the current official name. |
| 3 | `number-change` | Cited document number / year / code is outdated or malformed. |
| 4 | `abolished` | Cited document has been explicitly abolished. |
| 5 | `superseded` | Cited document has been replaced by a newer version. |

The Skill produces a **candidate issue list**. It never produces a final
audit conclusion without explicit human confirmation.

---

## When to Use

Use this Skill when the user provides a corporate policy / standard /
regulation document and asks to:

- audit / check / verify the reference list,
- check whether referenced regulations are still in force,
- check whether document numbers are still correct,
- detect missing references,
- review a policy before publication / revision,
- perform periodic annual compliance review.

Do **not** use this Skill when:

- the input is not a structured policy / standard / regulation document,
- the user only wants general legal advice without document inspection,
- the user wants a final audit opinion without a human reviewer in the loop.

---

## Inputs

The Skill expects:

| Input | Required | Format | Notes |
|-------|----------|--------|-------|
| Policy text | yes | plain text, markdown, or PDF | Extract references and body mentions. |
| Internal reference catalog (parent-company) | optional | xlsx / csv | Provided by deployment environment. Path declared in `Resources`. |
| Internal reference catalog (in-house) | optional | xlsx / csv | Provided by deployment environment. Path declared in `Resources`. |

If the optional catalogs are not present in the deployment environment,
the Skill should request them from the user before performing
internal-policy verification.

> **Internal reference catalogs are deployment-environment resources.**
> This public repository does not contain any proprietary corporate catalog.
> The Skill is designed to consume them as pluggable inputs.

---

## Outputs

The Skill produces:

1. **Reference extraction table** — every reference found in the document
   (name, number, location, classification).
2. **Implicit-reference table** — every regulation / standard / policy name
   mentioned in the body, even if not in the formal reference list.
3. **Candidate issue list** — every candidate issue, classified into the
   five categories.
4. **Final structured report** (after human confirmation) — an Excel or
   Markdown table containing the confirmed issues.

### Standard issue-record fields

Each candidate issue is recorded with:

| Field | Required | Meaning |
|-------|----------|---------|
| `index` | yes | Serial number, 1-based. |
| `standard_name` | yes | Full name and document number of the policy being audited. |
| `clause_location` | yes | Section / clause where the issue occurs. |
| `ref_entry` | yes | The reference line as it currently appears (name + number). |
| `issue_type` | yes | One of the five categories. |
| `description` | yes | What's wrong + potential impact. |
| `recommendation` | yes | Concrete fix wording. |
| `evidence` | recommended | Source of the finding (search result, catalog row, etc.). |
| `confidence` | recommended | `high` / `medium` / `low`. |
| `confirmation` | recommended | `pending` / `confirmed` / `rejected`. |

---

## Execution Rules

These rules are non-negotiable. Violating any of them is an execution failure.

### Must do

- **Extract first, verify second.** Extraction and verification are
  separate stages. Do not verify something that has not been extracted.
- **Classify every reference.** Before verification, classify each
  reference into one of the six classes (A: national law, B: regulation /
  departmental rule, C: local regulation, D: parent-company policy, E:
  in-house policy, S: standard).
- **Run all required queries for external references.** For each A/B/C/S
  class reference, run **all four** queries (current status, revision /
  supersession, abolition, document number). Running only one query and
  declaring "no result" is an execution failure.
- **Run all four steps for internal references.** For each D/E class
  reference, run all four steps (number match → name match → keyword fuzzy
  match → abolition). Skipping step 4 because step 1–3 produced nothing is
  an execution failure.
- **Record every candidate issue as it is found.** Do not wait until the
  end. Do not silently drop uncertain items.
- **Pause at every human confirmation gate.** See the `Human Confirmation`
  section.

### Must not

- **Do not declare "currently in force" without verification.** "No search
  result" is **not** equivalent to "currently in force".
- **Do not web-search internal policy numbers.** D/E class numbers are
  internal codes; web search returns nothing useful.
- **Do not skip the four-query rule for A/B/C/S.** Running 1–2 queries and
  stopping is an execution failure.
- **Do not auto-produce the final report.** The Skill must pause at every
  gate and wait for human confirmation.
- **Do not silently swallow uncertain candidates.** If uncertain,
  record the candidate and mark `confirmation: pending`.

### Pause conditions

The Skill must pause when:

- extraction is complete → human confirmation gate 1,
- candidate issue list is complete → human confirmation gate 2,
- a reference cannot be classified (A/B/C/D/E/S ambiguous) → user query.

### Non-confirmation conditions

The Skill must **not** declare a final conclusion when:

- search results are contradictory,
- search results are empty (for A/B/C/S),
- internal catalog file is missing (for D/E),
- classification is ambiguous.

In any of these cases, record the candidate, mark
`confirmation: pending`, and request human review.

---

## Workflow

```
Stage 1: Material reading
        ↓
Stage 2: Reference extraction
        ↓
Stage 3: Classification
        ↓
Stage 4: Verification (rule-based + tool-assisted)
        ↓
Stage 5: Issue recording
        ↓
Stage 6: Human Confirmation Gate  ← pause and wait
        ↓
Stage 7: Structured Output
```

### Step 1 — Material reading

Read the policy text in full. Identify the document's structure: sections,
clauses, formal reference list, and any annexes.

### Step 2 — Reference extraction

Extract from the document two kinds of references:

- **Formal references** — items listed in the formal reference section
  (titles like "规范性引用文件", "引用文件", "引用标准", etc.).
- **Implicit references** — regulations / standards / policies mentioned
  in the body text (e.g. inside `《》` brackets, or following phrases like
  "按照", "依据", "符合", "执行", "参照").

Record each reference with:

- name,
- document number (if present),
- clause location,
- source type (formal / implicit).

### Step 3 — Classification

Classify each reference into one of:

- **A — National laws** (e.g. 《XX法》)
- **B — Administrative regulations / departmental rules**
- **C — Local regulations / policies**
- **D — Parent-company policies** (internal, cross-organization)
- **E — In-house policies** (internal, this organization)
- **S — Standards** (national / industry / enterprise)

Classification drives the verification strategy in Step 4.

### Step 4 — Verification

#### 4.1 A / B / C / S classes — external references

Use a regulation retrieval tool. For each reference, run **all four**
queries:

1. `"{name}"` — current validity.
2. `"{name}"` — revision / supersession / latest version.
3. `"{name}"` — abolition status.
4. `"{document number}"` — number correctness.

Decision rules:

- 4-of-4 confirm current name and number → no issue.
- Result shows official new name → `name-change`.
- Result shows new document number → `number-change`.
- Result shows the document is abolished → `abolished`.
- Result shows the document is replaced by a new one → `superseded`.
- 4-of-4 return no result → **still record a candidate**, mark
  `confirmation: pending`, and ask the user to verify manually.
- Any contradiction among results → record the most likely candidate,
  mark `confirmation: pending`.

> **"No search result" does not automatically mean the document is
> invalid.** Record the item for human confirmation.

#### 4.2 D / E classes — internal references

**Do not web-search.** Use only the internal catalogs supplied by the
deployment environment. Run all four steps:

1. **Exact number match** — search the catalog's number column.
2. **Exact name match** — search the catalog's name column.
3. **Keyword fuzzy match** — strip suffixes like
   "管理办法" / "管理规定" / "办法" / "规定" / "细则" / "指引" and search
   the keywords.
4. **Abolition decision** — if steps 1–3 all fail, judge
   `abolished`.

The internal catalog is the **authoritative source** for D/E classes.
Absence in the catalog is meaningful: it means either the name has
changed (caught in step 3) or the document is abolished (step 4). There
is **no "not found but still in force" outcome** for D/E.

### Step 5 — Issue recording

For every verification outcome that is not "no issue", record a
candidate issue using the standard issue-record fields above. Record as
the verification proceeds — do not batch at the end.

### Step 6 — Human Confirmation Gate

**Pause.** Present the candidate issue list to the user, grouped by
issue category. Ask for confirmation. Do not auto-proceed.

If the user requests changes, adjust the candidate list, re-present, and
wait for confirmation again.

### Step 7 — Structured Output

After confirmation, produce the final structured report:

- Markdown table in the chat, **or**
- Excel file with the seven columns specified by `verification_rules.md`,
  **or** both.

---

## Verification Rules

See [`verification_rules.md`](verification_rules.md) for the complete
verification rules, including:

- the five issue categories with detection rules,
- the D/E four-step internal verification procedure,
- the search-result decision matrix,
- the "needs human confirmation" usage rules,
- the Excel output format and styling.

---

## Human Confirmation

The Skill enforces **two mandatory human confirmation gates**:

| Gate | After | Purpose | What to show |
|------|-------|---------|--------------|
| **Gate 1** | Step 3 (classification complete) | Confirm extraction and classification are complete. | Formal reference list + implicit reference list. |
| **Gate 2** | Step 5 (issue recording complete) | Confirm the candidate issue list is correct. | Candidate issues grouped by category, with counts. |

The Agent must:

1. stop after each gate,
2. display the relevant artifact,
3. explicitly ask the user to confirm,
4. wait for the user to reply with a confirmation keyword
   (`确认`, `继续`, `continue`, `proceed`, etc.),
5. only then proceed to the next stage.

If the user provides modification instructions, the Agent must adjust
the artifact, re-display it, and wait again. The Agent must not assume
that a "modification made" implies confirmation.

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Input is empty or unreadable | Ask user to provide a readable document. |
| No formal reference section found | Note that the document lacks a formal reference list; continue with implicit-reference detection only. |
| Reference classification is ambiguous | Ask the user. |
| Internal catalog file is missing | Ask the user to provide it, or note that D/E verification is partial. |
| Regulation retrieval tool returns errors | Record candidate with `confidence: low`, mark `confirmation: pending`. |
| Search results contradict each other | Record the most likely candidate, mark `confirmation: pending`. |

---

## Output Format

The final output is a structured table with at least:

```
| # | Document | Clause | Reference | Issue Type | Description | Recommendation | Confidence | Confirmation |
```

Excel version adds:

- merged title row,
- header row with light-blue background,
- issue-type column highlighted.

See [`verification_rules.md`](verification_rules.md) § 6 for full
Excel styling rules.

---

## Resources

| Resource | Source | Purpose |
|----------|--------|---------|
| Regulation / standard database | Public (e.g. 北法宝, 同花顺 iFinD) | A/B/C/S verification |
| `internal_standard_catalog.xlsx` | Deployment environment | D-class verification (parent-company policies) |
| `internal_policy_catalog.xlsx` | Deployment environment | E-class verification (in-house policies) |
| [`verification_rules.md`](verification_rules.md) | This repository | Full verification rules |

> **Internal reference catalogs are optional resources supplied by the
> deployment environment. This public repository does not contain any
> proprietary corporate catalog.**
