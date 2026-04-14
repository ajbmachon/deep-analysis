# deep-analysis

Verified code intelligence for Claude Code. Orchestrates multi-wave analysis with trust-scored reports.

## What it does

Dispatches specialized agents across multiple waves to analyze codebases, then independently verifies every claim:

1. **Wave 1 — Overview**: Broad structural mapping of the codebase
2. **Wave 2 — Deep-Dive**: Targeted, exhaustive analysis of specific areas
3. **Wave 3 — Verification**: Independent fact-checking of all claims against source code
4. **Wave 4 — Synthesis**: Merged final report with trust scores

## Components

| Component | Type | Purpose |
|-----------|------|---------|
| `deep-analysis` | Skill | Orchestrator — manages waves, sizes work packages, synthesizes reports |
| `deep-code-auditor` | Agent | Exhaustive static analysis with file:line evidence trails |
| `doc-verifier` | Agent | Forensic verification of written claims against actual source code |

## Installation

```bash
claude --plugin-dir /path/to/deep-analysis
```

Or add to your project's `.claude/settings.json`:
```json
{
  "plugins": ["/path/to/deep-analysis"]
}
```

## Usage

Trigger the skill with any of these phrases:

- "Deeply analyze src/core/domain/"
- "Audit the authentication module with verification"
- "I need a reliable analysis of the order processing code"
- "Analyze and verify the payment gateway"

The skill will:
1. Size the target and propose a wave plan
2. Dispatch overview agents (Wave 1)
3. Create work packages from overview findings
4. Dispatch deep-dive agents (Wave 2)
5. Dispatch verification agents (Wave 3)
6. Synthesize a final report with trust scores (Wave 4)

## Output

All artifacts are written to `.ai/` in the project root:

```
.ai/
├── wave1/{slug}/          # Structural overview reports
│   ├── audit.log
│   └── overview-report.md
├── wave2/{slug}/          # Deep-dive analysis reports
│   ├── audit.log
│   └── analysis-report.md
├── verification/          # Verification reports
│   └── verification-{slug}-{date}.md
└── final/                 # Synthesized final report
    └── deep-analysis-{date}.md
```

## Trust Scores

Every report includes a trust score: `verified / (verified + contradicted) * 100%`. This tells you how reliable the analysis is. Unverifiable claims are excluded from the calculation.

## Requirements

- Claude Code with plugin support
- LSP server configured for the target language (optional but recommended for better analysis)

## License

MIT
