# Final Report Template

Write the synthesized report to `.ai/final/deep-analysis-{date}.md` using this structure:

```markdown
# Deep Analysis Report — {target description}

**Date:** {YYYY-MM-DD}
**Git commit:** {hash}
**Targets analyzed:** {list}
**Wave structure:** {N} overview agents -> {M} deep-dive agents -> {P} verifiers
**Trust score:** {X}% ({verified}/{total} claims verified)

## Executive Summary

[3-5 bullet points of the most important findings across all waves.
Flag any contradicted claims — these are things the analysis got wrong.]

## Architecture Overview

[From Wave 1 — the structural map, key patterns, organization principles]

## Per-Package Findings

### {Package 1 name}

**Verification:** {N} verified | {N} contradicted | {N} unverifiable

#### Key Findings
[Top findings from Wave 2, annotated with verification status]

#### Contradictions Found
[Anything Wave 3 flagged as wrong — quote both the claim and the correction]

#### Technical Debt
[Verified tech debt items only]

### {Package 2 name}
[repeat structure]

## Cross-Cutting Themes

[Patterns across multiple packages: shared dependencies, common coupling,
repeated tech debt patterns, architectural concerns]

## Recommendations

[Actionable next steps, prioritized. Reference specific files and line numbers
from the verified analysis.]

## Appendix

### Wave 1 Reports
- .ai/wave1/{slug}/overview-report.md

### Wave 2 Reports
- .ai/wave2/{slug}/analysis-report.md (per package)

### Verification Reports
- .ai/verification/verification-{slug}-{date}.md (per package)
```

## Trust Score Calculation

`verified_claims / (verified_claims + contradicted_claims) * 100`, ignoring unverifiable claims. Report both the percentage and the raw numbers so the user can judge for themselves.
