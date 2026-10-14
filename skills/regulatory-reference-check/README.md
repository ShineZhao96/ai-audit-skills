# regulatory-reference-check

Standardized Reference Document Verification Skill.

## What it does

Inspects a corporate policy / standard / regulation document and detects
candidate issues in its reference list across five disjoint categories:

- `missing-reference`
- `name-change`
- `number-change`
- `abolished`
- `superseded`

The Skill extracts every formal reference and every implicit body
reference, classifies them into A/B/C/D/E/S classes, applies class-specific
verification rules, and produces a candidate issue list that **must be
reviewed by a human** before any final report is generated.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The Skill definition itself: description, inputs, outputs, workflow, execution rules, human confirmation gates, error handling. |
| `verification_rules.md` | The rule book: detection rules per category, D/E four-step procedure, search-result decision matrix, output formatting. |
| `examples/example_input.md` | A synthetic short policy fragment used as a runnable example. |
| `examples/example_output.md` | The expected candidate issue list for the example input. |

## Quick start

1. Load `SKILL.md` into your Agent runtime.
2. Provide the policy text the user wants to audit.
3. Provide internal reference catalogs (D / E classes) if available.
4. Run the workflow; the Agent will pause for human confirmation at
   Gate 1 (after extraction) and Gate 2 (after issue recording).

## Required deployment-environment resources

The Skill does not bundle any proprietary catalog. To verify D / E
classes (parent-company and in-house policies), the deployment
environment must provide:

| Resource | Format |
|----------|--------|
| `internal_standard_catalog.xlsx` | Parent-company policy catalog. Columns: `number`, `name`, `issue_date`, `effective_date`. |
| `internal_policy_catalog.xlsx` | In-house policy catalog. Columns: `number`, `name`, `issue_date`, `effective_date`. |

If either catalog is missing, the Agent should request it from the
user before performing D / E verification.

## Limits

- External verification depends on having a regulation-retrieval
  connector available. The Skill does not bundle one.
- No private data, document number, or proprietary catalog is
  embedded in this Skill. See the top-level `README.md` Privacy &
  Disclaimer.

## Related

- Top-level: [`README.md`](../../README.md)
- Methodology: [`docs/methodology.md`](../../docs/methodology.md)
- Sibling Skill: [`responsibility-audit`](../responsibility-audit/)
