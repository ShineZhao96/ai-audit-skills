# Example Output (Synthetic)

> **Illustrative Example / Synthetic Data.**
> The output below is what the Skill is expected to produce for the
> input in `example_input.md`. It is synthetic and intended for
> methodology illustration only.

The output below assumes the Agent has access to:

- a regulation retrieval tool that returns search results,
- a synthetic parent-company catalog `internal_standard_catalog.xlsx`,
- a synthetic in-house catalog `internal_policy_catalog.xlsx`.

---

## Step 1 — Extraction (Gate 1)

### Formal references (from the formal reference section)

| # | Reference name | Document number |
|---|----------------|-----------------|
| 1 | 《中华人民共和国招标投标法》 | (none) |
| 2 | 《中华人民共和国政府采购法》 | (none) |
| 3 | GB/T 19001-2008《质量管理体系 要求》 | GB/T 19001-2008 |
| 4 | 《示例采购管理办法》 | 示例文号 |
| 5 | 《示例供应商管理细则》 | (none) |

### Implicit references (from body text)

| Body location | Implicit reference |
|---------------|--------------------|
| 第一条 | "国家相关法律法规" |
| 第五条 | 《示例采购管理办法》（already in formal list, partial duplicate) |
| 第六条 | "相关法律法规" + "本企业内部采购工作指引" |
| 第八条 | 《供应商管理实施细则》(name differs from formal list item 5) |

### Reverse-lookup from catalogs

| Catalog row | Body match |
|-------------|------------|
| 《示例采购管理办法》 Q/EX-PROC-2023-007 | Article 5 mentions it |
| 《示例供应商管理细则》 Q/EX-SUP-2022-014 | Article 8 mentions 《供应商管理实施细则》 (different name) |

---

## Step 2 — Classification

| # | Reference | Class | Notes |
|---|-----------|-------|-------|
| 1 | 《中华人民共和国招标投标法》 | A | National law. |
| 2 | 《中华人民共和国政府采购法》 | A | National law. |
| 3 | GB/T 19001-2008 | S | Standard. |
| 4 | 《示例采购管理办法》（示例文号） | E | In-house policy. Document number is a placeholder. |
| 5 | 《示例供应商管理细则》 | E | In-house policy. No number listed. |
| 6 | 《供应商管理实施细则》 (implicit) | E | In-house policy. Different name from #5. |

---

## Step 3 — Verification (per class)

### A — National law

GB/T 19001-2008 has been superseded by GB/T 19001-2016. So reference 3
is a `number-change` and also a `superseded`. (For brevity, this
example picks one: `superseded`.)

### E — In-house policy

- Reference 4: catalog contains 《示例采购管理办法》 Q/EX-PROC-2023-007.
  Formal list shows "示例文号" — `number-change`.
- Reference 5: catalog contains 《示例供应商管理细则》
  Q/EX-SUP-2022-014. Formal list omits the number — `number-change`.
- Implicit reference 6: body uses 《供应商管理实施细则》; catalog's
  exact name is 《示例供应商管理细则》 → fuzzy-match
  succeeds → `name-change`.

### Implicit missing-reference

- Body article 6 mentions "本企业内部采购工作指引" — not in formal
  list → `missing-reference`.

---

## Step 4 — Candidate issue list (Gate 2)

| # | Document | Clause | Reference | Issue Type | Description | Recommendation | Confidence | Confirmation |
|---|----------|--------|-----------|------------|-------------|----------------|------------|--------------|
| 1 | 《示例企业采购管理制度》Q/EX-PROC-2024-001 | 附录-规范性引用文件 第3项 | GB/T 19001-2008《质量管理体系 要求》 | superseded | 该标准已被 GB/T 19001-2016 整体替代；可能造成引用版本过时，体系审核依据失效。 | 将 GB/T 19001-2008 修改为 GB/T 19001-2016《质量管理体系 要求》。 | high | pending |
| 2 | 《示例企业采购管理制度》Q/EX-PROC-2024-001 | 附录-规范性引用文件 第4项 | 《示例采购管理办法》（示例文号） | number-change | 引用文号为占位符；可能造成引用无法定位。 | 修改为：《示例采购管理办法》（Q/EX-PROC-2023-007）。 | high | pending |
| 3 | 《示例企业采购管理制度》Q/EX-PROC-2024-001 | 附录-规范性引用文件 第5项 | 《示例供应商管理细则》（文号未列） | number-change | 引用未列文号；可能造成引用定位困难。 | 修改为：《示例供应商管理细则》（Q/EX-SUP-2022-014）。 | high | pending |
| 4 | 《示例企业采购管理制度》Q/EX-PROC-2024-001 | 第八条 | 《供应商管理实施细则》 | name-change | 该制度目录中现行名称为《示例供应商管理细则》；可能造成与现行制度不一致。 | 修改为：《示例供应商管理细则》（Q/EX-SUP-2022-014）。 | high | pending |
| 5 | 《示例企业采购管理制度》Q/EX-PROC-2024-001 | 第六条 | （未列出的《本企业内部采购工作指引》） | missing-reference | 正文提及但未在引用章节列出；可能造成执行缺乏依据。 | 在规范性引用文件中增加《本企业内部采购工作指引》及文号。 | medium | pending |

---

## Step 5 — Human Confirmation Gate

The Agent must now stop and present the candidate list above to the
human reviewer. After the reviewer confirms (or modifies), the Agent
produces the final structured report (Excel or final Markdown).

If the reviewer accepts all five:

- Issue 1 (`superseded`): confirmed.
- Issue 2 (`number-change`): confirmed.
- Issue 3 (`number-change`): confirmed.
- Issue 4 (`name-change`): confirmed.
- Issue 5 (`missing-reference`): confirmed.

The reviewer can also reject any row (e.g. if Issue 1 is judged
acceptable to leave as historical reference).

---

## Step 6 — Final structured report (after confirmation)

After confirmation, the Agent produces the final structured report
in the same table format, with `confirmation` set to `confirmed` /
`rejected` per row. Excel formatting follows
`verification_rules.md` § 10.2.
