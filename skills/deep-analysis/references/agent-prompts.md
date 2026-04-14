# Agent Prompt Templates

## Wave 1 — Overview Agent Prompt

Use `subagent_type: "deep-code-auditor"`. Split the target into 2-5 overview agents by logical area.

```
Perform a STRUCTURAL OVERVIEW of: {target_path}

You are a Wave 1 (overview) agent. Your job is to MAP THE LANDSCAPE, not analyze every file.

Focus on:
1. Directory structure and organization patterns
2. Key interfaces, abstract classes, and base types (read their signatures, not implementations)
3. Design patterns used (CQRS, Strategy, Factory, Builder, etc.)
4. Naming conventions and how consistently they're followed
5. Complexity hotspots — which subdirectories have the most files/classes?
6. Cross-cutting dependencies — which types are imported across many directories?
7. Technical debt signals — inconsistencies, legacy patterns, missing abstractions

Do NOT:
- Read every file line by line
- Analyze implementation details of every method
- Produce file-by-file breakdowns

USE LSP to accelerate your work (if available):
- `documentSymbol` on key files to get class/method inventories without reading every line
- `workspaceSymbol` to find all classes matching patterns (e.g., all *Interface classes)
- `goToImplementation` on key interfaces to count implementations
- `findReferences` on heavily-used types to gauge coupling
- Use LSP on at least 10-15 key files/symbols

DELIVERABLE — identify areas for Wave 2 deep-dive:
At the end of your report, include a section "## Recommended Deep-Dive Targets" listing
specific directories/files that need detailed Wave 2 analysis, with a brief reason why.
Group them into suggested work packages of ~20-40 files each.

Write your outputs to:
- Audit log: .ai/wave1/{slug}/audit.log
- Overview report: .ai/wave1/{slug}/overview-report.md

Current git commit: {commit_hash}
Today's date: {date}

IMPORTANT — Lean return protocol:
Return ONLY a short status summary. Do NOT include the full report in your response.

Return format:
STATUS: complete | partial | failed
FILES: [list of file paths you wrote]
SUMMARY: [1-2 sentences — key structural finding]
DEEP_DIVE_TARGETS: [list the recommended Wave 2 work packages, one per line]
ISSUES: [only if status is not complete]
```

## Wave 2 — Deep-Dive Agent Prompt

Use `subagent_type: "deep-code-auditor"`. Dispatch all packages **in parallel**.

```
Perform a DETAILED DEEP-DIVE analysis of these specific files/directories:

{exact_list_of_directories_and_files}

You are a Wave 2 (deep-dive) agent. A Wave 1 overview has already mapped the broad structure.
Here is the relevant Wave 1 context for your area (paste the section of the Wave 1
overview that covers this package's area, typically 10-30 lines):

--- WAVE 1 CONTEXT ---
{relevant_wave1_section}
--- END WAVE 1 CONTEXT ---

Your job is EXHAUSTIVE DETAIL for this specific package:

1. **Every public interface and class**: Document purpose, key methods, constructor dependencies
2. **Design patterns**: How this package implements patterns (CQRS, Builder, Strategy, etc.)
3. **Value objects & type safety**: What constraints are enforced? What validation happens?
4. **Exception handling**: What exceptions can be thrown? Are they specific or generic?
5. **Dependencies**: What does this package import from other packages? Map the coupling.
6. **Naming convention compliance**: Does every class follow the project's naming rules?
7. **Technical debt**: Missing tests, inconsistent patterns, legacy workarounds, TODO comments
8. **Edge cases**: Null handling, empty collections, boundary conditions

USE LSP extensively (if available):
- `documentSymbol` on every file to get complete class/method inventory
- `goToImplementation` on every interface to find concrete implementations
- `findReferences` on key types to map who uses them
- `incomingCalls` / `outgoingCalls` on critical methods to trace execution flow
- `hover` on type hints for full qualified class names and docblocks
- Use LSP on EVERY file in your package — this is a deep-dive, not an overview

Write your outputs to:
- Audit log: .ai/wave2/{slug}/audit.log
- Analysis report: .ai/wave2/{slug}/analysis-report.md

Current git commit: {commit_hash}
Today's date: {date}

IMPORTANT — Lean return protocol:
Return ONLY a short status summary. Do NOT include the full report in your response.

Return format:
STATUS: complete | partial | failed
FILES: [list of file paths you wrote]
SUMMARY: [1-2 sentences — key finding]
CLAIMS_COUNT: [approximate number of verifiable claims in your report]
ISSUES: [only if status is not complete]
```

## Wave 3 — Verification Agent Prompt

Use `subagent_type: "doc-verifier"`. Dispatch all in parallel.

```
Verify the following code analysis report against the actual source code.
Be skeptical — check every claim about file paths, function signatures, dependencies,
and behavioral descriptions. Extract all verifiable claims from the analysis report,
then verify each one independently.

Documents to verify:
- Analysis report: .ai/wave2/{slug}/analysis-report.md
- Supporting evidence log: .ai/wave2/{slug}/audit.log

For each claim, classify as:
- VERIFIED: claim is accurate per current source code
- CONTRADICTED: claim is wrong — state what's actually true
- UNVERIFIABLE: cannot confirm or deny (e.g., runtime behavior claim)

USE LSP to verify efficiently (if available):
- `documentSymbol` to check if claimed methods/properties exist
- `goToDefinition` to verify type references
- `findReferences` to check claimed coupling/usage patterns
- `hover` to verify type signatures and docblocks

Write your verification report to: .ai/verification/verification-{slug}-{date}.md

IMPORTANT — Lean return protocol:
Return ONLY a short status summary. Do NOT include the full report in your response.

Return format:
STATUS: complete | partial | failed
FILES: [list of file paths you wrote]
VERIFIED: [count]
CONTRADICTED: [count]
UNVERIFIABLE: [count]
SUMMARY: [1-2 sentences]
ISSUES: [only if status is not complete]
```
