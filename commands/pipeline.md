---
description: Run full kiro spec-driven pipeline with quality gates and audit trail
allowed-tools: Read, SlashCommand, TodoWrite, Bash, Write, Glob, Task
argument-hint: ["feature description" | feature-name] [--from=phase] [--sequential]
---

# Kiro Pipeline

Run the full kiro spec-driven lifecycle for a feature with quality gates, audit trail, and automatic implementation mode selection.

## Variables

INPUT: $ARGUMENTS

## Pre-flight

1. Load the `kiro:pipeline-orchestration` skill for pipeline phases, quality gates, and domain knowledge.
2. Parse INPUT:
   - **Quoted string** -> new feature description. Start at Phase 1 (Init).
   - **Matches `.kiro/specs/{name}/`** -> existing spec. Read `spec.json` for current phase.
   - **`--from=` flag** -> override starting phase (requirements, design, tasks, impl, validation).
   - **`--sequential` flag** -> force sequential implementation mode.
3. If existing spec: read `pipeline-log.md` (if exists) for context on prior decisions.

## Execute

Follow the kiro:pipeline-orchestration skill phases in order, starting from the detected/specified phase.

**At each phase:**
1. Invoke the corresponding kiro skill
2. Run the quality gate checks defined in the skill
3. Present results to the user for approval
4. Append to `pipeline-log.md` with timestamp, metrics, and decision
5. Proceed to next phase only after approval

**At Phase 6 (Implementation):**
- Invoke `/kiro:spec-impl {feature}`
- If `--sequential` was set or no parallel `(P)` markers exist, standard sequential mode
- If parallel streams exist, note them in the log for user awareness

## Report

After each phase completion:
- Current phase and status
- Quality gate result (PASS/FAIL)
- Next phase and what it will do
- Link to `pipeline-log.md` for full audit trail

After final validation:
- Summary of all phases completed
- Test results and coverage numbers
- Total time from init to validation
- Recommendation for next steps (commit, PR, deploy)
