# AI Audit Skills

> **这是什么**：企业制度里的法规引用经常过期、错名、已废止，部门与岗位职责也常出现缺失、重叠、层级错配。人工逐条比对成本高、漏检风险大。
>
> **怎么做**：把「提取 → 分类 → 规则校验 → 人工确认 → 结构化输出」固化成可复用的 Agent Skill，校验规则显式可查，不依赖模型“觉得对”。
>
> **产出什么**：两类问题清单（引用有效性 5 类、职责审计 7 类）和可直接进入底稿的结构化报告；每个 Skill 在给出结论前强制暂停，等待人工确认。

> **Reusable AI Agent Skills for Compliance Review and Organizational Responsibility Audit**

A collection of reusable AI Agent Skills abstracted from real-world enterprise
compliance, internal control, and organizational responsibility auditing practice.
This repository is intended for engineers, auditors, and AI builders who want to
study how a complex audit workflow can be packaged into a structured, verifiable,
human-in-the-loop Agent Skill — and who want to reuse those patterns in their own
scenarios.

---

## Overview

Internal compliance review and organizational responsibility audit are
labor-intensive, evidence-driven tasks that traditionally require senior staff to
cross-check many documents, cross-reference regulations, and reconcile
overlapping responsibilities across departments and positions.

`ai-audit-skills` distills two such workflows into reusable AI Agent Skills:

1. **`regulatory-reference-check`** — verifies that the references cited inside a
   corporate policy/standard/regulation document are still valid, current, and
   complete.
2. **`responsibility-audit`** — performs a bidirectional audit between
   department-level responsibilities and position-level responsibilities,
   surfacing gaps, overlaps, conflicts, and ambiguous wording.

Both skills are designed to be run by an Agent, with strict human confirmation
gateways that the AI must respect before producing final reports.

---

## Skills at a Glance

| Skill | 用途 |
|---|---|
| [`regulatory-reference-check`](skills/regulatory-reference-check/SKILL.md) | 检查制度文件中的法规及内部文件引用 |
| [`responsibility-audit`](skills/responsibility-audit/SKILL.md) | 审计部门、岗位职责及二者之间的映射关系 |

> **🚀 Demo：** [查看完整执行案例](docs/demo.md)

---

## Why This Project

Conventional AI audit attempts tend to stop at a single-round prompt or simple
text comparison, which makes it difficult to cover a complete review workflow.
Without explicit extraction, classification, verification, issue-recording, and
human-review stages, the output may look authoritative but is hard to examine,
trace, or defend under review.

This project demonstrates an alternative architecture:

```
Prompt
  ↓
Workflow
  ↓
Rules
  ↓
Verification
  ↓
Human Review
  ↓
Structured Output
```

Each Skill in this repo is structured so that the Agent first extracts facts
from the input, classifies them, applies explicit verification rules, records
candidate issues, **pauses for human confirmation**, and only then produces a
structured final report.

This pattern is portable. Once you understand it, you can apply it to many
other audit and review scenarios.

---

## Skills

### 1. `regulatory-reference-check`

**Standardized Reference Document Verification**

Inspects the reference list of a corporate policy / standard / regulation
document and detects five **reference-validity** issue categories
(scope: citation checking only — independent of the seven responsibility
categories used by `responsibility-audit` below):

| # | Issue | Meaning |
|---|-------|---------|
| 1 | 应引未引 (Missing reference) | Body refers to a regulation but reference list omits it |
| 2 | 名称变更 (Name change) | Cited name no longer matches the current official name |
| 3 | 文号变更 (Document-number change) | Cited document number / year / code is outdated |
| 4 | 已废止 (Abolished) | Cited document has been explicitly abolished |
| 5 | 已替代 (Superseded) | Cited document has been replaced by a newer one |

Workflow:
`Material reading → Reference extraction → Classification → Cross-checking →
Validity / change / abolition / supersession verification → Issue recording →
Human confirmation → Structured output`

Internal references (e.g. parent-company policies, in-house standards) are
verified against optional internal catalogs. External regulations are verified
through regulation-retrieval tools.

See [`skills/regulatory-reference-check/SKILL.md`](skills/regulatory-reference-check/SKILL.md).

### 2. `responsibility-audit`

**Department ↔ Position Responsibility Bidirectional Audit (Three-Step Method)**

Cross-checks department-level responsibility descriptions against
position-level (job) responsibility descriptions to expose seven
**responsibility** issue categories:

| # | Issue | Direction |
|---|-------|-----------|
| 1 | 空白缺陷 (Gap) | Department responsibility without position counterpart, or vice versa |
| 2 | 职责重复 (Overlap) | Same responsibility claimed by multiple positions |
| 3 | 冲突交叉 (Conflict) | Two positions or departments claim the same scope |
| 4 | 模糊歧义 (Ambiguity) | Wording too vague to execute or assess |
| 5 | 详略不当 (Granularity mismatch) | One level is much more detailed than the other |
| 6 | 表述不规范 (Non-standard wording) | Colloquial verbs, vague quantifiers, etc. |
| 7 | 层级不匹配 (Layer mismatch) | Department "does" while position "organizes", etc. |

Workflow is strictly per-department, three steps:
1. Department responsibility review
2. Position responsibility review
3. Bidirectional cross-check between department and position

Each step pauses for human confirmation before proceeding to the next.

See [`skills/responsibility-audit/SKILL.md`](skills/responsibility-audit/SKILL.md).

---

## Field Usage (Desensitized)

- **Context**: internal-control review at the procurement center of an energy
  group — policy inventory, contract and ledger compliance checks; roughly two
  months (Jul–Aug 2026).
- **What changed**: the "policy requirement ↔ responsibility list" comparison
  moved from manual line-by-line reading to "Agent extracts elements and
  citations → rules verify → human confirms".
- **Three kinds of issues the human gate actually caught**:
  1. Boundary cases the rules did not cover — the same matter falls under
     different clauses depending on the procurement method.
  2. The same wording meaning different things at department vs. position level;
     the model tends to judge them as consistent.
  3. Misjudging "abolished / superseded" — the cited document number exists, but
     its validity status has already changed.
- **Note**: `docs/demo.md` uses synthetic data. No real project material, policy
  text, or client information is included in this repository.

---

## 🚀 Demo

通过一个完整的模拟案例，展示 AI Audit Skills 如何完成：

**输入 → 提取 → 分类 → 验证 → 问题发现 → 人工确认 → 结构化输出**

👉 [查看完整 Demo](docs/demo.md)

The Demo contains two end-to-end runs:

| Demo | Skill | What it shows |
|------|-------|---------------|
| **Demo A** | `regulatory-reference-check` | Synthetic policy text → reference extraction → **Gate 1 (extraction complete)** → A/B/C/D/E/S classification → external 4-query / internal 4-step verification → 4 candidate issues → **Gate 2 (3 confirmed + 1 rejected, incl. a rejection example)** → structured output |
| **Demo B** | `responsibility-audit` | The three-step method: department review → position review → bidirectional cross-check, with coverage statistics |

Each Demo is structured around five layers, so a reviewer can see not just
*what* the Skill outputs, but *why*:

```text
Business Problem → AI Workflow → Rule-based Verification
        → Human Confirmation → Structured Output
```

> All Demo content is **synthetic**, labeled
> `Illustrative Example / Synthetic Data`. It demonstrates method and
> workflow — it does not publish business data.

---

## Architecture

Both skills share the same overall architecture:

```
Input (text / table / PDF)
      ↓
Stage 1: Extraction
      ↓
Stage 2: Classification
      ↓
Stage 3: Verification (rule-based + tool-assisted)
      ↓
Stage 4: Issue Recording
      ↓
Stage 5: Human Confirmation Gate  ← required pause
      ↓
Stage 6: Structured Output (Markdown / Excel)
```

Key design decisions:

- **Separation of extraction and verification.** The Agent first extracts
  facts (references, clauses, responsibilities), then runs verification in a
  second pass. This avoids the common failure mode where the model "verifies"
  something it has just invented.
- **Rule-based verification rules.** Each issue category has an explicit
  detection rule documented in `verification_rules.md`. This makes the
  verification auditable.
- **Two mandatory human confirmation gates.** The AI never produces a final
  report without explicit human review. This is essential because compliance
  judgments cannot be fully delegated to a model.
- **Optional internal resources.** Internal reference catalogs and policy
  registries are referenced but **not bundled** in this public repo. They are
  deployment-environment resources.

---

## Workflow

A typical run of `regulatory-reference-check` looks like:

1. User provides the policy text (or PDF) to be audited.
2. Agent extracts every reference plus every implicit reference in the body.
3. Agent classifies each reference (national law, regulation, local rule,
   standard, parent-company policy, in-house policy).
4. Agent applies the verification rules for each category.
5. Agent records every candidate issue (with confidence and source).
6. **Pause — user reviews the candidate issue list.**
7. After confirmation, Agent produces the final structured report.

The `responsibility-audit` skill follows the same pattern, but on a
per-department basis and with three sub-steps inside the
"verification → issue recording" phase.

---

## Human-in-the-Loop Design

This project deliberately **does not** aim for a fully autonomous audit. Two
hard constraints apply:

1. **The AI must not skip extraction or verification steps.** A reference that
   was not extracted cannot be verified, and a fact that was not verified
   cannot appear in the final report.
2. **The AI must pause for human confirmation at every gate.** No final
   output is produced without an explicit "continue" from the user.

Why? Because:

- Compliance judgments are legal/regulatory and require accountability.
- Public regulation databases are incomplete; search results cannot be
  treated as ground truth.
- Internal catalogs reflect organizational context that the model does not
  have.
- AI models hallucinate, and audit conclusions are exactly where hallucination
  is most damaging.

The AI's role is to **find candidate issues, organize evidence, and propose
language for fixes**. The human's role is to **confirm which candidates are
real, and which fix language to adopt**.

---

## Quick Examples

Each Skill ships a **minimal** runnable example in its own `examples/` directory:

- `example_input.md` — a synthetic short policy / department table, clearly
  labeled as illustrative.
- `example_output.md` — the expected issue list the Skill would produce, also
  synthetic.

Use these to sanity-check a Skill in one pass.

For the **full end-to-end walkthrough** — including the verification reasoning
chain and the human confirmation gate — see the [Demo](docs/demo.md).

> All examples in this repository are **Illustrative Example / Synthetic Data**.
> They do not represent any real organization.

---

## Project Structure

```
ai-audit-skills/
├── README.md
├── LICENSE
├── .gitignore
├── skills/
│   ├── regulatory-reference-check/
│   │   ├── SKILL.md
│   │   ├── verification_rules.md
│   │   ├── README.md
│   │   └── examples/
│   │       ├── example_input.md
│   │       └── example_output.md
│   │
│   └── responsibility-audit/
│       ├── SKILL.md
│       ├── README.md
│       └── examples/
│           ├── example_input.md
│           └── example_output.md
│
└── docs/
    ├── methodology.md
    └── demo.md          ← 完整端到端案例 Demo
```

---

## Design Principles

- **Reality over presentation.** This repository documents patterns that
  actually worked in audit work, not polished marketing material.
- **No proprietary data.** Internal catalogs, document numbers, and company
  names have been generalized. Deployment environments must provide their
  own catalogs.
- **No false accuracy claims.** No "X% accuracy" or "Y× efficiency" numbers
  are quoted. Such numbers require a benchmark that this public repo does
  not have.
- **Explicit rules.** Every issue category has an explicit detection rule,
  documented separately.
- **Human confirmation is non-negotiable.** The Skill refuses to produce a
  final report without it.

---

## Limitations

- **Internal catalogs are not bundled.** This repo does not (and legally
  cannot) include any real corporate reference catalog. To use the
  `regulatory-reference-check` Skill on internal documents, the deployment
  environment must provide its own catalogs.
- **Regulation retrieval requires external tooling.** External verification
  depends on having access to a reliable regulation / standard database.
  Pluggable via connector.
- **The Skills are guides, not products.** They are designed to be loaded by
  an Agent runtime that supports file reading, tool calling, and user
  confirmation. They are not a turnkey SaaS.
- **Language is bilingual-friendly.** The Skills are written with bilingual
  headings (Chinese + English). The Agent should answer in the user's
  language.

---

## Provenance

The rules and Skills in this repository are desensitized from an internal-control
consulting project the author took part in. Original project materials, policy
texts, client and vendor information are not published; all examples in this
repository are synthetic.

Division of work: business-rule analysis came from the actual work in that
internship; the abstraction into Skills, documentation, testing and iteration were
done individually. This repository contains no code, documents or data belonging
to any employer.

---

## Privacy & Disclaimer

This repository is published for **methodology demonstration**. The Skills
encode patterns and verification rules, not corporate data.

- No real organization name, employee name, customer name, document number,
  or proprietary catalog appears in this repo.
- All examples are synthetic. They are clearly labeled
  `Illustrative Example / Synthetic Data`.
- The Skills are **not legal advice** and should not be used as a substitute
  for professional compliance review.

By using these Skills, you agree that:

- You are responsible for providing your own internal catalogs.
- You are responsible for the final compliance judgment — the AI only
  surfaces candidates.
- You will not redistribute any internal data through this project.

---

## Future Improvements

Potential extensions (not promised; just possible directions):

- A third Skill: **采购流程合规审查 (Procurement-process compliance review)**.
- Pluggable connectors for different regulation databases (北法宝, 同花顺iFinD,
  etc.).
- A small test harness with synthetic inputs to regression-test the
  Skills' rules.
- Optional CSV export schema for direct integration with existing audit
  management systems.

---

## License

MIT. See [`LICENSE`](LICENSE).
