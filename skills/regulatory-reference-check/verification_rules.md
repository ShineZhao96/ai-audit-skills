# Verification Rules

> Reference verification rules for the `regulatory-reference-check` Skill.

This document defines the rule book that the Agent applies during
Step 4 (Verification) and Step 5 (Issue Recording). It is intentionally
written as rules, not narrative — so that the verification logic can be
audited and extended.

---

## 1. Purpose

Provide a deterministic, auditable rule set that an Agent can apply to
decide:

- whether a cited reference is currently valid,
- whether its name or number matches the current official version,
- whether the cited document has been abolished or superseded,
- whether the body text mentions a reference that is missing from the
  formal reference list,
- whether the item should be flagged for human review.

The rules are **class-specific**. External references and internal
references use different verification strategies.

---

## 2. Issue Categories

All issues must be classified into exactly one of these five categories.
The categories are disjoint and exhaustive for this Skill's scope.

| Code | Issue | Detection rule |
|------|-------|----------------|
| `missing-reference` | 应引未引 | Body mentions `《XX》` (or via "按照/依据/符合/执行/参照") but the formal reference list omits it. |
| `name-change` | 名称变更 | Formal reference's name does not match the current official name. |
| `number-change` | 文号变更 | Formal reference's document number / year / code does not match the current official number; or the number format is malformed (e.g. double hyphen). |
| `abolished` | 已废止 | Formal reference has been explicitly abolished and has no replacement. |
| `superseded` | 已替代 | Formal reference has been replaced by a newer version or document. |

If a finding does not fit one of these five, it is **out of scope** for
this Skill. Do not invent a sixth category.

---

## 3. Reference Classification

Every reference is classified into exactly one of:

| Code | Class | Verification strategy |
|------|-------|------------------------|
| A | 国家法律 (national law) | Tool: regulation retrieval. |
| B | 行政法规 / 部门规章 (administrative / departmental rule) | Tool: regulation retrieval. |
| C | 地方性法规 / 政策 (local regulation / policy) | Tool: regulation retrieval. |
| D | 上级公司制度 (parent-company policy) | Catalog: `internal_standard_catalog`. |
| E | 本公司制度 (in-house policy) | Catalog: `internal_policy_catalog`. |
| S | 国家 / 行业 / 企业标准 (standard) | Tool: regulation retrieval (year validation). |

If a reference spans multiple classes (e.g. a local implementation
rule of a national law), pick the **highest-level** class for
verification purposes, and record the secondary class in
`description`.

---

## 4. Verification Strategy

### 4.1 External references (A / B / C / S)

For each reference, run **all four** queries:

1. `"{name}"` — current validity.
2. `"{name}"` — revision / supersession / latest version.
3. `"{name}"` — abolition status.
4. `"{document number}"` — number correctness.

Running 1–2 queries and stopping is an execution failure.

### 4.2 Internal references (D / E)

For each reference, run **all four steps** in the internal catalog:

1. **Exact number match** — search the catalog's number column.
2. **Exact name match** — search the catalog's name column.
3. **Keyword fuzzy match** — strip suffixes like
   "管理办法" / "管理规定" / "办法" / "规定" / "细则" / "指引", then
   search keywords.
4. **Abolition decision** — if steps 1–3 all fail, judge `abolished`.

Skipping step 4 because step 1–3 produced nothing is an execution failure.

---

## 5. Internal Reference Verification (D / E Classes)

### 5.1 Why no web search

D / E references use internal document codes (e.g.
`<org-code>-<category>-<seq>-<year>`). These are not findable through
public web search. Web searching D / E wastes time and returns nothing
useful.

### 5.2 Step 1 — Exact number match

Search the catalog's number column for an exact match.

- **Match** → go to step 2 with the matched row.
- **Partial match** (e.g. cited number is missing the year) → record
  the complete number from the catalog. Issue type: `number-change`.
- **No match** → go to step 2 using the original number.

### 5.3 Step 2 — Exact name match

Search the catalog's name column for the reference's name verbatim.

- **Match, name identical, number identical** → no issue.
- **Match, name identical, number differs** → issue type:
  `number-change`. Use the catalog's number.
- **No match** → go to step 3.

### 5.4 Step 3 — Keyword fuzzy match

Strip common suffix words: 管理办法, 管理规定, 办法, 规定, 细则,
指引, 制度, 规程, 规范, 标准.

Then search the catalog's name column for entries containing the
remaining keywords.

- **Match** → issue type: `name-change`. The cited name has changed;
  the catalog's name is the current official name.
- **No match** → go to step 4.

### 5.5 Step 4 — Abolition decision

If steps 1–3 all fail, the reference is judged `abolished`. The
catalog is the **authoritative source** for D / E; absence in the
catalog is meaningful and is itself the verdict.

**Description wording** for `abolished`:

> 经查，该制度未在现行有效制度目录中出现，确认已废止。

The description **must not** include phrases like
"经查上级公司制度目录" or "经查本公司制度目录" or
"经查知识库" — the agent does not know which catalog the user is
actually using. Use neutral "经查" wording.

### 5.6 D / E must not use "needs human confirmation"

For D / E classes, the catalog is the authoritative source. There is
no "needs human confirmation" outcome:

- catalog match → issue found with high confidence;
- catalog miss → `abolished`.

A D / E candidate with `confirmation: pending` is an execution error.

---

## 6. External Regulation Verification (A / B / C / S Classes)

### 6.1 Search-result decision matrix

| Search result | Decision |
|---------------|----------|
| Latest official version, name and number match | No issue |
| Latest official version, name differs | `name-change` |
| Latest official version, number differs | `number-change` |
| Document abolished (no replacement) | `abolished` |
| Document superseded by a newer one | `superseded` |
| All 4 queries return no result | **Still record a candidate**. Issue type: most likely category. Description starts with `【需人工确认】`. |
| Search results contradict each other | **Still record the most likely candidate**. Description starts with `【需人工确认】`. |
| Body mentions, formal list omits | `missing-reference` |

### 6.2 Recommended queries

For A / B / C / S references, run these four queries:

1. `"{name}"` — current validity.
2. `"{name}" 修订 OR 替代` — revision / supersession.
3. `"{name}" 废止` — abolition.
4. `"{number}"` — number validation.

### 6.3 "No result" is not a clean bill

> **No search result does not automatically mean that a document is
> invalid. Record the item for manual confirmation.**

When all 4 queries return no result:

- Do record the candidate (do not silently drop).
- Issue type: most likely category (e.g. `abolished` if the user's
  context suggests it, else `name-change`).
- Description: must begin with `【需人工确认】`, e.g.
  > 【需人工确认】经查询，法律法规检索工具未检索到该法规，无法确认
  > 现行有效性，建议人工核实。
- `confirmation: pending`.

---

## 7. Search Result Decision Rules

When multiple search results disagree:

- Prefer the **most recent** official source.
- Prefer a publisher-of-record source over third-party summaries.
- If still ambiguous, record the most likely candidate and mark
  `confirmation: pending`.

When search results are partial (e.g. only number is confirmed):

- Use the partial confirmation to upgrade `confidence`.
- Still record the candidate if any sub-rule is not satisfied.

---

## 8. Manual Confirmation Rules

Use `confirmation: pending` only when:

- A / B / C / S: search results are empty or contradictory.
- A / B / C / S: classification is ambiguous.
- D / E: catalog file is **missing** (not the same as "no match").
- Document number format is malformed but the intended number is not
  clear.

Do **not** use `confirmation: pending` when:

- D / E: catalog returned no match → must judge `abolished`.
- Issue is clear-cut (e.g. body mentions but list omits).

---

## 9. Issue Recording Rules

### 9.1 Required fields per record

| Field | Required | Notes |
|-------|----------|-------|
| `index` | yes | 1-based serial; gaps are not allowed. |
| `standard_name` | yes | Full name and document number of the policy being audited. Even if the same policy has multiple issues, never leave this field empty. |
| `clause_location` | yes | Section / clause number. E.g. `2 规范性引用文件`, `5.3 项目立项`. |
| `ref_entry` | yes | The reference as currently cited (name + number). |
| `issue_type` | yes | One of the five codes above. |
| `description` | yes | What's wrong + potential impact. |
| `recommendation` | yes | Concrete fix wording. Must be copy-pasteable. |
| `evidence` | recommended | Source: search result summary, catalog row, etc. |
| `confidence` | recommended | `high` / `medium` / `low`. |
| `confirmation` | recommended | `pending` / `confirmed` / `rejected`. |

### 9.2 Description style

- **First sentence:** what's wrong, with quoted original text.
- **Second sentence:** potential impact in practice.
- **Length:** keep consistent with other entries of the same audit run.

Examples:

> 该制度标题中包含未在引用章节列出的合规要求；
> 可能导致日常执行中遗漏合规依据。

> 该制度编号中年号格式有误（使用了双连字符 `--`）；
> 可能导致检索时无法精确匹配到最新版本。

### 9.3 Recommendation style

- Use explicit modification verbs: `修改为：`, `增加条款：`, `删除：`.
- Provide copy-pasteable wording.
- Never use vague phrases like "建议完善", "建议补充",
  "需进一步明确".

### 9.4 Confidence and confirmation defaults

| Outcome | `confidence` | `confirmation` |
|---------|--------------|----------------|
| A / B / C / S, search confirms issue | `high` | `pending` |
| A / B / C / S, search returns no result | `medium` | `pending` |
| D / E, exact match → issue | `high` | `pending` |
| D / E, fuzzy match → `name-change` | `high` | `pending` |
| D / E, no match → `abolished` | `high` | `pending` |

`pending` is the default at record time. The human confirmation gate
updates it to `confirmed` or `rejected`.

---

## 10. Output Requirements

### 10.1 Markdown output

Markdown output is a single table:

```
| # | Document | Clause | Reference | Issue Type | Description | Recommendation | Confidence | Confirmation |
```

### 10.2 Excel output

| Column | Header | Notes |
|--------|--------|-------|
| A | 编号 | 1-based serial. |
| B | 涉及管理标准 | Full name + document number; required for every row. |
| C | 涉及条款/位置 | Section / clause number. |
| D | 规范性引用文件条目 | Original reference as cited. |
| E | 问题分类 | One of the five codes (highlighted cell). |
| F | 问题描述 | Description (see 9.2). |
| G | 完善建议 | Recommendation (see 9.3). |

Excel styling:

- Row 1: merged title, e.g.
  `<Org Name> 管理标准中规范性引用文件核查成果清单`.
- Row 2: header row, bold, light-blue background.
- Issue-type column: orange background, bold.
- Wrap text; freeze header (data starts row 3); enable auto-filter.

### 10.3 Output gating

The Skill **must not** produce an Excel / final Markdown output until
after the second human confirmation gate has been cleared. A
preliminary Markdown table for review is acceptable; the final output
is not.

---

## 11. Anonymized Example

### 11.1 Input (synthetic)

```
《示例企业采购管理制度》

第五条  采购活动应遵守《示例采购管理办法》（示例文号）。
第六条  采购人员应按照相关法律法规开展采购活动。
```

### 11.2 Candidate issues

```
| # | Document | Clause | Reference | Issue Type | Description | Recommendation | Confidence | Confirmation |
|---|----------|--------|-----------|------------|-------------|----------------|------------|--------------|
| 1 | 《示例企业采购管理制度》 | 第五条 | 《示例采购管理办法》（示例文号） | number-change | 引用文号未填写完整；可能导致引用无法定位。 | 补充完整文号，例如《示例采购管理办法》（Q/EX-XXX-XX-XXXX）。 | medium | pending |
| 2 | 《示例企业采购管理制度》 | 第六条 | （未列出的相关法律法规） | missing-reference | 正文提到"相关法律法规"但未在引用章节列出具体法规；可能导致执行缺乏依据。 | 在规范性引用文件中补充具体适用法律法规名称及文号。 | high | pending |
```

### 11.3 Notes

- Both rows require human confirmation before final output.
- Row 1 may be rejected by the reviewer if the policy author intentionally
  uses a placeholder number pending issuance.
- Row 2 is a typical `missing-reference` finding.
