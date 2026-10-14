---
name: responsibility-audit
description: "Department ↔ Position Responsibility Bidirectional Audit (Three-Step Method). Triggers: 部门职责与岗位职责双向校核 / 部门职责审核 / 岗位职责审核 / 三步审核法 / 部门职责VS岗位职责 / 职责一致性检查 / 职责空白 / 职责重叠 / 职责冲突."
---

# responsibility-audit

> Department ↔ Position Responsibility Bidirectional Audit (Three-Step Method)

---

## Description

Cross-checks department-level responsibility descriptions against
position-level (job) responsibility descriptions within an organization,
surfacing gaps, overlaps, conflicts, ambiguities, granularity mismatches,
non-standard wording, and layer mismatches.

The Skill follows a strict **three-step method**, applied per department,
with a mandatory human confirmation gate between steps:

1. **Step 1** — Department responsibility review.
2. **Step 2** — Position responsibility review.
3. **Step 3** — Bidirectional cross-check (department ↔ position).

Each step produces a structured Markdown table. After all three steps
are confirmed by the human reviewer, the audit produces a consolidated
report covering all three steps.

---

## When to Use

Use this Skill when the user asks to:

- audit department responsibilities against position responsibilities,
- perform a three-step responsibility audit,
- cross-check department vs. position tables,
- detect responsibility gaps, overlaps, or conflicts,
- harmonize wording across department and position documents,
- prepare a responsibility system for revision.

Do **not** use this Skill when:

- only department responsibilities are available (no position data) —
  use Step 1 only and flag the missing data.
- only position responsibilities are available — use Step 2 only.
- the user wants a full text rewrite rather than an issue list.

---

## Inputs

The Skill expects:

| Input | Required | Format | Notes |
|-------|----------|--------|-------|
| Department responsibility table | yes | xlsx / csv / markdown | One row per department; columns typically include `department`, `clause`, `responsibility`. |
| Position responsibility table | yes | xlsx / csv / markdown | One row per position; columns typically include `department`, `position`, `clause`, `responsibility`. |
| Work-standard PDFs (per position) | optional | PDF | Used in Step 3 to resolve ambiguity. Provided by the deployment environment. |
| Senior-executive (高管) work-standard PDFs | optional | PDF | Used for senior-executive Step 2 review. |

If the work-standard PDFs are not available, the Agent must rely on
the position responsibility table only, and flag uncertainty in the
output.

> **All inputs are deployment-environment resources.** This public
> repository does not bundle any real organizational data. The example
> in `examples/` is synthetic.

---

## Outputs

The Skill produces:

1. **Step 1 table** — issues found in department responsibilities.
2. **Step 2 table** — issues found in position responsibilities.
3. **Step 3 table** — bidirectional cross-check issues.
4. **Consolidated table** (after all three confirmations) — all issues
   with continuous serial numbering.
5. **Per-step statistics** — counts per step.

### Standard issue-record fields

| Field | Required | Meaning |
|-------|----------|---------|
| `index` | yes | Serial, 1-based, continuous across all three steps. |
| `department` | yes | The audited department; for senior executives, fill with `高管`. |
| `dept_clause` | conditional | Reference to the department-responsibility clause (if relevant). |
| `position_clause` | conditional | Reference to the position-responsibility clause or work-standard clause (if relevant). |
| `issue_type` | yes | One of the seven categories below. |
| `root_cause` | yes | Where the problem originates — `department-responsibility` / `position-responsibility` / `both`. |
| `description` | yes | What's wrong + potential impact; both parts present. |
| `recommendation` | yes | Concrete fix wording; copy-pasteable; self-contained. |

---

## Execution Rules

These rules are non-negotiable.

### Must do

- **Process strictly step-by-step per department.** Do not skip a step
  or merge multiple steps into one pass.
- **Use semantic matching, not literal matching.** Two clauses may be
  worded differently but mean the same thing. Recognize semantic
  equivalence before flagging an issue.
- **Cite original text verbatim.** Every issue must quote the relevant
  clause text. Do not paraphrase.
- **Issue self-contained recommendations.** A recommendation must be
  copy-pasteable into the target document without referring to other
  rows ("见上条", "已在第X项中一并解决", etc. are forbidden).
- **Pause at every gate.** See the `Human Confirmation` section.

### Must not

- **Do not skip departments or merge them.** Each department is audited
  separately.
- **Do not bulk-process multiple departments in one pass.** Doing so
  hides per-department issues and breaks the gating logic.
- **Do not judge "inconsistent" purely from wording.** Open the work
  standard PDF or position text to verify before recording.
- **Do not auto-produce the consolidated report.** Wait for human
  confirmation.

### Pause conditions

The Skill must pause when:

- Step 1 for a department is complete → confirm before Step 2.
- Step 2 for a department is complete → confirm before Step 3.
- All three steps for a department are complete → confirm before
  moving to the next department.
- A position responsibility is ambiguous → open the work-standard PDF.

### Non-confirmation conditions

The Skill must **not** declare a finding final when:

- semantic equivalence cannot be determined without the work-standard
  PDF (record as `confidence: low` and request the PDF),
- the same wording appears in both department and position but
  semantics are unclear (record as candidate, mark `pending`).

---

## Workflow

```
For each department (one at a time):

  Step 1: Department Responsibility Review
      ↓
   Pause — confirm
      ↓
  Step 2: Position Responsibility Review
      ↓
   Pause — confirm
      ↓
  Step 3: Bidirectional Cross-Check
      ↓
   Pause — confirm
      ↓
  Move to next department

After all departments:
  Consolidate into one final table.
```

---

### Step 1 — Department Responsibility Review

Check department-level responsibilities for:

1. **空白缺陷 (Gap)** — important responsibility missing from the
   department's clauses.
2. **冲突交叉 (Conflict)** — boundary with another department is
   unclear.
3. **模糊歧义 (Ambiguity)** — wording is too vague to execute or
   assess.
4. **表述不规范 (Non-standard wording)** — colloquial verbs ("做好",
   "抓好"), loose catch-all clauses ("其他工作"), or layer-inverted
   verbs.
5. **职责重复 (Overlap)** — same responsibility claimed by two
   departments.
6. **详略不当 (Granularity mismatch)** — one department describes a
   function in one sentence while others use a hundred characters.

Output: a table with the eight standard fields (see Outputs).

### Step 2 — Position Responsibility Review

Check position-level responsibilities for:

1. **职责重复 (Overlap)** — multiple positions claim the same
   responsibility.
2. **冲突交叉 (Conflict)** — boundary between positions is unclear.
3. **详略不当 (Granularity mismatch)** — same-level positions have
   very different detail.
4. **模糊歧义 (Ambiguity)** — vague wording.
5. **表述不规范 (Non-standard wording)** — colloquial verbs, etc.
6. **层级不匹配 (Layer mismatch)** — deputy is a mechanical copy of
   the principal.

> Note: Step 2 also applies to **senior executives (高管)** — they have
> no department responsibilities, so only Step 2 is performed.

Output: same eight-field table.

### Step 3 — Bidirectional Cross-Check

This is the most valuable step. For each department, perform **two
directional checks**:

- **Department → Position**: for each department clause, find the
  position(s) responsible for it. If no position carries it → `gap`.
- **Position → Department**: for each position clause, find a
  department-level counterpart. If none → potential scope creep or
  misplacement.

Cover all seven issue categories at this step:

| # | Issue | Direction |
|---|-------|-----------|
| 1 | gap | department→position or position→department |
| 2 | overlap | department→position or position→department |
| 3 | conflict | department↔position |
| 4 | ambiguity | department→position |
| 5 | granularity | department↔position |
| 6 | non-standard | department↔position |
| 7 | layer-mismatch | department↔position |

If semantic equivalence cannot be determined without the work-standard
PDF, open it via the relevant connector.

Output: same eight-field table, plus a coverage statistic (number /
percentage of positions affected per gap row).

---

## Issue Categories

The Skill recognizes exactly seven issue categories. They are
disjoint and exhaustive for this audit goal.

| Code | Issue | Definition |
|------|-------|------------|
| `gap` | 空白缺陷 | Either: a department responsibility has no position counterpart; or a position responsibility has no department counterpart. |
| `overlap` | 职责重复 | Two or more entities (departments or positions) carry the same responsibility. |
| `conflict` | 冲突交叉 | Two entities claim the same scope with different intent; boundaries unclear. |
| `ambiguity` | 模糊歧义 | Wording is too vague to execute or assess. |
| `granularity` | 详略不当 | Detail-level mismatch between two layers. |
| `non-standard` | 表述不规范 | Colloquial verbs, loose catch-alls, or other non-standard wording. |
| `layer-mismatch` | 层级不匹配 | Deputy is a mechanical copy of the principal; verbs are inverted between department and position. |

---

## Human Confirmation

Three mandatory gates per department:

| Gate | After | Purpose | Display |
|------|-------|---------|---------|
| **Gate 1** | Step 1 | Confirm department-responsibility findings. | Step 1 table. |
| **Gate 2** | Step 2 | Confirm position-responsibility findings. | Step 2 table. |
| **Gate 3** | Step 3 | Confirm bidirectional cross-check findings. | Step 3 table with coverage stats. |

After Gate 3, the Agent moves to the next department. After the last
department, the Agent consolidates and presents the final report. A
fourth implicit gate (consolidation review) is recommended but not
required.

If the user provides modification instructions at any gate, the Agent
must adjust, re-display, and wait again. The Agent must not assume
that "modification made" equals confirmation.

---

## Output Format

### Per-step Markdown table

Eight-column Markdown table:

```
| # | Department | Dept Clause | Position Clause | Issue Type | Root Cause | Description | Recommendation |
```

### Per-step statistics

After the table, a small statistics block:

```
| Step | Items |
|------|-------|
| Step 1: Department Responsibility Review | N items |
| Step 2: Position Responsibility Review | N items |
| Step 3: Bidirectional Cross-Check | N items |
| Total | N items |
```

### Consolidated report

After all departments are confirmed, a single table with continuous
serial numbering and the same per-step statistics at the end.

### Optional Excel output

The same table can be exported to Excel. Styling:

- Row 1: merged title `<Org Name> 部门职责与岗位职责双向校核成果清单`.
- Row 2: header row, bold, light-blue background.
- Issue-type column: orange background, bold.
- Wrap text; freeze header (data starts row 3); enable auto-filter.

---

## Resources

| Resource | Source | Purpose |
|----------|--------|---------|
| Department responsibility table | Deployment environment | Step 1 input. |
| Position responsibility table | Deployment environment | Step 2 input. |
| Work-standard PDFs (per position) | Deployment environment | Step 3 disambiguation. |
| Senior-executive work-standard PDFs | Deployment environment | Senior-executive Step 2. |
| [`../../docs/methodology.md`](../../docs/methodology.md) | This repository | Methodology background. |

> **All real organizational data is supplied by the deployment
> environment.** This public repository does not contain any
> proprietary corporate catalog, position description, or work-standard
> document.
