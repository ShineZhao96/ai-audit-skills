# Methodology

This document explains the design philosophy behind the Skills in this
repository. It is meant for readers who want to understand **why the Skills
are structured the way they are**, so they can adapt the pattern to their
own audit / review / verification workflows.

---

## 1. Why AI Skills, not just Prompts

A "prompt" is a single instruction. An **AI Skill** is an instruction plus:

- a structured input contract,
- an explicit multi-stage workflow,
- a rule book the Agent must apply,
- a verification strategy,
- a structured output contract,
- and (importantly) **mandatory human confirmation gates**.

In audit and compliance scenarios, prompts alone are not enough. They give
no way to:

- separate fact extraction from judgment,
- apply a verifiable rule book,
- prevent the model from "verifying" something it just invented,
- ensure a human sees the candidate issues before they become a report.

So this project packages each scenario as a Skill. A Skill can be loaded by
an Agent runtime (e.g. WorkBuddy, Claude, custom agent frameworks) and
executed step-by-step.

The generic pattern is:

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

---

## 2. The generic structure of an audit-style AI task

Most audit-style AI tasks reduce to the same skeleton, even when the
domain changes:

```
Input
  ↓
Extraction
  ↓
Classification
  ↓
Verification
  ↓
Issue Detection
  ↓
Human Confirmation
  ↓
Structured Output
```

### 2.1 Input

The Agent receives a structured input: a text policy, an Excel sheet, a
PDF, or a combination. The Skill's `Inputs` section declares exactly what
is expected.

### 2.2 Extraction

The Agent extracts facts from the input: list of references, list of
responsibilities, list of clauses. Extraction is performed in a single
pass and produces a structured artifact (e.g. a list of items with
location pointers).

**Critical:** Extraction must be **separate** from verification. If the
Agent extracts and verifies in the same step, it tends to "verify" things
it has just invented.

### 2.3 Classification

Each extracted item is classified. For example:

- reference type: national law / regulation / local rule / standard /
  parent-company policy / in-house policy,
- responsibility level: department / position.

Classification enables different verification strategies per class. In
`regulatory-reference-check`, external regulations are searched via a
tool, while internal policies are checked against an internal catalog.

### 2.4 Verification

Verification is rule-based. Each issue category has an explicit
detection rule documented separately (e.g.
`skills/regulatory-reference-check/verification_rules.md`).

Verification is also **defensive**:

- Search tools may return no result. "No result" ≠ "currently valid".
- Internal catalogs are authoritative; absence is significant.
- Any uncertain item is recorded as a candidate, not silently dropped.

### 2.5 Issue Detection

Every candidate issue is recorded with:

- issue type (one of the predefined categories),
- entity (the document / clause / position),
- original text (quoted verbatim),
- description (what's wrong + potential impact),
- recommendation (concrete fix wording),
- evidence / reference,
- confidence,
- human confirmation status.

This is the minimum information needed for a human reviewer to make a
judgment.

### 2.6 Human Confirmation

The Agent must pause and present the candidate issues to the human
reviewer. **No final report is produced without explicit confirmation.**

The pause is enforced by the Skill itself, not by the model's good
behavior. The Skill's `Execution Rules` section states:

> AI must stop and present the candidate issues. After the user
> explicitly confirms (e.g. "确认", "continue"), the Agent proceeds to
> produce the structured final report.

### 2.7 Structured Output

After confirmation, the Agent produces a structured output: a Markdown
table, an Excel file, or both. The structure is fixed by the Skill, so
that downstream tooling (audit management systems, spreadsheets, BI
dashboards) can ingest it predictably.

---

## 3. Why Human-in-the-Loop is non-negotiable

Three reasons, each independent:

### 3.1 Legal and regulatory accountability

Compliance judgments are legal acts. Whoever signs the final report
bears the responsibility. The AI cannot bear legal responsibility.
Therefore the AI cannot sign off.

### 3.2 Incompleteness of public data

Public regulation databases are useful but not exhaustive. A regulation
may exist but not be indexed by the tool. A local regulation may not be
publicly available. An internal policy obviously is not.

So an absent search result **cannot** be interpreted as "this regulation
is no longer in force". The only honest interpretation is: "we cannot
confirm; please review".

### 3.3 Hallucination

Models hallucinate. In audit contexts, a hallucinated compliance
conclusion is exactly the failure mode that causes the most damage. The
human confirmation gate is the last line of defense against
hallucination reaching a downstream decision-maker.

---

## 4. The five category philosophy

In `regulatory-reference-check`, all detected issues fall into exactly
five categories:

1. **应引未引** (Missing reference) — body refers to, list omits.
2. **名称变更** (Name change) — cited, but the official name has changed.
3. **文号变更** (Document-number change) — cited, but the number is wrong.
4. **已废止** (Abolished) — cited, but the document is explicitly abolished.
5. **已替代** (Superseded) — cited, but a newer version exists.

Why exactly five? Because they are **disjoint** and **exhaustive**
relative to the audit goal. Adding more categories creates ambiguity
without adding detection power. A new failure mode should be analyzed
first to determine whether it fits one of the five, or whether the
audit goal itself needs to be redefined.

The same principle applies to `responsibility-audit`'s seven issue
categories. They are not a wish list; they are the categories that
actually occur and that have distinct detection rules.

---

## 5. The bidirectional audit pattern

`responsibility-audit` introduces a pattern that is reusable far beyond
responsibilities:

> If A is a higher-level description and B is a lower-level description
> of the same domain, audit by checking **both directions**:
>
> - Does every item in A have a counterpart in B? (coverage gap)
> - Does every item in B have a counterpart in A? (scope creep)

This bidirectional pattern is robust against a single-direction
blind spot. In one direction, missing coverage is invisible. In the
other direction, unauthorized scope is invisible. Only the intersection
sees both.

---

## 6. Generic reference architecture

The Skills reference external resources, but **do not bundle them**:

| Resource | Source |
|----------|--------|
| Regulation / standard database | Public (北法宝, 同花顺iFinD, etc.) |
| Internal reference catalog | Provided by deployment environment |
| Internal policy archive | Provided by deployment environment |

This separation is critical for two reasons:

1. **Public repos cannot legally contain proprietary catalogs.**
2. **The Skill should work in any deployment environment.** A
   deployment-specific catalog should be pluggable, not hardcoded.

The Skill files include a `Resources` section that documents which
resources are expected, and which ones must be supplied by the
deployment environment.

---

## 7. How to extend this pattern

If you want to add a third Skill (e.g. procurement-process compliance
review), follow these steps:

1. Write the audit goal as a one-sentence problem statement.
2. Define the input contract: what exactly does the Agent receive?
3. Define the output contract: what exactly does the Agent produce?
4. Enumerate the disjoint, exhaustive issue categories.
5. For each category, write an explicit detection rule.
6. Decide where human confirmation is required.
7. Document the Skills in the same `SKILL.md` template used here.
8. Add a synthetic example to the `examples/` directory.
9. Update the top-level `README.md` to point at the new Skill.

If a step is unclear, **stop and redesign**. A Skill that cannot be
expressed in this structure probably should not be a Skill — it might
need a different architecture (e.g. an interactive assistant, a
report generator, or a dataset pipeline).
