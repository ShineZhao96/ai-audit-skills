# responsibility-audit

Department ↔ Position Responsibility Bidirectional Audit (Three-Step Method).

## What it does

Cross-checks department-level responsibility descriptions against
position-level (job) responsibility descriptions within an organization,
surfacing seven categories of issues:

- `gap` (空白缺陷)
- `overlap` (职责重复)
- `conflict` (冲突交叉)
- `ambiguity` (模糊歧义)
- `granularity` (详略不当)
- `non-standard` (表述不规范)
- `layer-mismatch` (层级不匹配)

The audit is **per-department, three-step**, with a mandatory human
confirmation gate between steps:

1. **Step 1** — Department responsibility review.
2. **Step 2** — Position responsibility review.
3. **Step 3** — Bidirectional cross-check.

After all three steps are confirmed, the Skill consolidates into one
final report.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The Skill definition itself. |
| `examples/example_input.md` | Synthetic department + position responsibility tables. |
| `examples/example_output.md` | The expected Step 1 / Step 2 / Step 3 outputs. |

## Quick start

1. Load `SKILL.md` into your Agent runtime.
2. Provide the department responsibility table and position
   responsibility table.
3. Process departments one at a time.
4. The Agent pauses after each step for human confirmation.
5. After all departments and confirmations, the Agent produces the
   consolidated report.

## Required deployment-environment resources

The Skill does not bundle any real organizational data. To run a real
audit, the deployment environment must provide:

| Resource | Format |
|----------|--------|
| Department responsibility table | xlsx / csv / markdown |
| Position responsibility table | xlsx / csv / markdown |
| Work-standard PDFs | PDF (per position, per department) |
| Senior-executive work-standard PDFs | PDF |

## Limits

- The Skill assumes departments and positions have stable IDs that can
  be referenced across tables. If the deployment environment uses
  different IDs, the Agent must build a mapping first.
- Without work-standard PDFs, Step 3 cannot resolve semantic
  ambiguity. The Agent will record candidates with `confidence: low`.
- Senior executives (高管) have no department responsibilities, so
  only Step 2 applies to them.

## Related

- Top-level: [`README.md`](../../README.md)
- Methodology: [`docs/methodology.md`](../../docs/methodology.md)
- Sibling Skill: [`regulatory-reference-check`](../regulatory-reference-check/)
