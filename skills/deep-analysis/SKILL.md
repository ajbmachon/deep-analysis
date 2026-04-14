---
name: "deep-analysis"
description: "This skill should be used when the user asks to deeply analyze code, audit a module or codebase, assess technical debt, understand architecture and dependencies in depth, or produce verified documentation about how code works. Trigger phrases include: 'deeply analyze', 'audit this code', 'analyze and verify', 'thorough code review', 'I need a reliable analysis', 'audit with proof', 'investigate the architecture', 'what does this code actually do', 'code review with evidence', 'analyze the codebase'. Orchestrates multi-wave analysis with independent verification and trust-scored reports."
---

# Deep Analysis — Verified Code Intelligence

You are an **orchestrator**. You do NOT read source code yourself. You dispatch specialized agents across multiple waves, manage their outputs, quality-assure results, and deliver a polished final report.

Two agent types do the real work:
- **deep-code-auditor** — static analysis with evidence trails (file:line references for every claim)
- **doc-verifier** — forensic verification of written claims against actual source code

## Multi-Wave Pipeline

```
Wave 0: Setup & Sizing
Wave 1: Overview (broad scope, structural mapping)
    | identifies areas of interest
Wave 2: Deep-Dive (narrow scope, detailed analysis per work package)
    | produces detailed claims
Wave 3: Verification (check Wave 2 claims against source)
    | produces trust scores
Wave 4: Synthesis & Promotion (merge into final report)
```

**Wave count depends on target size:**

| Target size (source files) | Waves needed |
|----------------------------|-------------|
| 1-30 files | 2 waves (analyze + verify) — skip Wave 1 |
| 31-200 files | 3 waves (overview + deep-dive + verify) |
| 200+ files | 4 waves (overview + deep-dive + verify + synthesize) |

## Wave 0 — Setup & Sizing

`$ARGUMENTS` contains the text the user provided after the trigger phrase. For example, if the user says "deeply analyze src/core/", then `$ARGUMENTS` is `src/core/`. Parse it for analysis target(s): files, directories, or multiple paths.

For each target, derive a **slug** for file naming (e.g., `product-domain`, `cart-adapter`). If the user provides no target, ask them what to analyze.

Get the git commit hash and create the working directory:
```bash
git rev-parse --short HEAD
mkdir -p .ai/wave1 .ai/wave2 .ai/verification .ai/final
```

**Size the target** to determine the wave plan:
```bash
find {target_path} -type f \( -name "*.php" -o -name "*.py" -o -name "*.ts" -o -name "*.js" -o -name "*.go" -o -name "*.rs" -o -name "*.java" \) | wc -l
```

Adjust the file extensions for the project's language. If above 30 files, get the subdirectory breakdown to plan work packages.

Present the sizing and wave plan to the user. Use TaskCreate to track each wave with dependencies.

## Wave 1 — Structural Overview

**Goal:** Map the landscape. Identify structure, patterns, complexity hotspots, and areas for deep-dive. NOT exhaustive — deliberately broad.

**When to skip:** Total target <=30 files — go directly to Wave 2.

Split the target into 2-5 overview agents by logical area. Read `references/agent-prompts.md` for the Wave 1 prompt template. Dispatch all agents in parallel.

### After Wave 1 Completes

1. **Quality gate** — Read the first ~40 lines of each overview report. Confirm structure and file references.
2. **Extract deep-dive targets** — Read the "Recommended Deep-Dive Targets" section. Collect suggested work packages.
3. **Create Wave 2 work packages** — Merge, deduplicate, and size-check (see rules below).
4. **Progress update** — Tell the user how many areas were mapped and how many packages identified.

## Wave 2 — Targeted Deep-Dive

**Goal:** Exhaustive, detailed analysis of areas identified by Wave 1. Small, focused work packages with file:line evidence.

### Work Package Sizing (CRITICAL)

| Package size | Quality expectation |
|-------------|-------------------|
| 5-15 files | Excellent — read every file, trace every dependency |
| 16-30 files | Good — read key files, LSP the rest |
| 31-40 files | Acceptable — must prioritize |
| 40+ files | **TOO LARGE — split further** |

Present the package manifest to the user before dispatching. Read `references/agent-prompts.md` for the Wave 2 prompt template. Dispatch all packages in parallel.

### After Wave 2 Completes

1. **Quality gate** — Read the first ~30 lines of each analysis report. Confirm substance.
2. **Collect claims counts** — Sum approximate verifiable claims across all packages.
3. **Progress update** — Report packages analyzed and claims to verify.

## Wave 3 — Verification

**Goal:** Independently verify Wave 2 claims against actual source code.

Each Wave 2 report gets one doc-verifier agent. If a report has >200 claims, split into two verifiers. Read `references/agent-prompts.md` for the Wave 3 prompt template. Dispatch all in parallel.

### After Wave 3 Completes

1. **Collect verdicts** — Calculate trust scores per package.
2. **Flag contradictions** — If any package has >20% contradicted claims, flag to user and offer to re-run.
3. **Progress update** — Report verified/contradicted/unverifiable counts and overall trust score.

## Wave 4 — Synthesis & Promotion

**Goal:** Merge all wave outputs into a single polished report. This is the one thing you write yourself.

Read each overview, deep-dive report, and verification report. Write the final report to `.ai/final/deep-analysis-{date}.md` following the template in `references/final-report-template.md`.

**Trust score:** `verified / (verified + contradicted) * 100`, ignoring unverifiable claims.

After presenting the report, list all files produced. Ask the user **one time** via AskUserQuestion:
- **Promote all to docs/** — copy `.ai/` tree, offer git commit
- **Keep in .ai/** — leave as-is
- **Delete .ai/** — clean up

## Failure Handling

- **Agent returns STATUS: failed** — Re-dispatch once with the same prompt. If it fails again, skip that package and note the gap in the final report.
- **Agent returns STATUS: partial** — Check which output files exist on disk. Use whatever was produced; note incomplete coverage in the final report.
- **Agent produces empty output files** — Treat as failed. Re-dispatch once.
- **Partial wave completion** (e.g., 3 of 5 agents succeed) — Proceed with available results. Note skipped packages. Do not block the entire pipeline on one failure.
- **Agent returns content inline instead of writing files** — Write the content to the expected file path yourself, then proceed normally.

## Rules

- **You are the orchestrator, not the analyst.** Do not read source code yourself. Read agent outputs.
- **Never skip verification.** Wave 3 is mandatory.
- **Respect context limits.** No agent gets >40 files for deep analysis. Split if larger.
- **Wave 1 feeds Wave 2.** Pass relevant Wave 1 context into Wave 2 prompts.
- **Lean returns only.** Agent prompts must include the lean return protocol.
- **Parallel within waves, sequential between waves.**
- **Track with Tasks.** Use TaskCreate for each wave, update as they complete.
- **Progress updates.** Report transitions between waves clearly.
- **One confirmation, at the end.** One AskUserQuestion after Wave 4.

For LSP usage guidance when composing agent prompts, read `references/lsp-guide.md`.
