---
name: spec-reviewer
description: Substantive critical review of kiro spec artifacts (requirements, design, tasks) against the actual codebase and quality standards. Produces GO/NO-GO verdicts with specific, actionable issues.
license: MIT
compatibility: Claude Code, Cursor, VS Code, Windsurf
metadata:
  category: methodology
  complexity: advanced
  author: Kiro Team
  version: "1.0.0"
---

# Spec Reviewer

Perform substantive critical review of kiro spec artifacts by cross-referencing against the actual codebase and project quality standards. Produces a GO/NO-GO decision with specific, actionable issues.

## Invocation

```
/kiro:reviewer {feature} requirements     # Review requirements.md
/kiro:reviewer {feature} design           # Review design.md
/kiro:reviewer {feature} tasks            # Review tasks.md
```

## Philosophy

This reviewer does what a senior engineer would do: read the spec, read the code, compare them, and identify gaps. Mechanical checks (EARS format, checkbox syntax) are necessary but insufficient. The real value is **substantive review** -- finding hollow sections, orphaned references, missing integrations, and misalignment between spec and reality.

## Review Workflow

```
1. Load Context
   |-- Read the target artifact
   |-- Read spec.json for feature metadata
   |-- Read pipeline-log.md for prior decisions (if exists)
   +-- Check for design mockup references in the spec description

2. Explore the Codebase
   |-- Identify target source files from the artifact
   |-- Read current implementations of referenced components
   |-- Inventory existing hooks, utilities, and data patterns
   +-- Map what EXISTS vs what the artifact ASSUMES exists

3. Apply Review Checklist (artifact-specific, see below)

4. Produce Verdict
   |-- GO: No high-severity issues, artifact is ready for next phase
   |-- NO-GO: High-severity issues that would cause problems downstream
   +-- Specific issues with severity, evidence, and fix guidance
```

## Review Checklists

### Requirements Review

| # | Check | Severity | What to Look For |
|---|-------|----------|------------------|
| R1 | Content depth | HIGH | Are all requirements substantive? A section with 3 or fewer ACs that duplicate existing functionality is hollow. Compare each requirement's AC count and richness against the actual components available. |
| R2 | Codebase alignment | HIGH | Do requirements reference components that exist? Do they MISS components that exist? Read the target files and their imports. If the codebase has existing components or hooks that the requirement ignores, that's a gap. |
| R3 | Orphaned references | MEDIUM | Do test requirements (unit/E2E) reference behaviors not specified in functional requirements? If a test requirement says "test authorization logic" but no functional requirement defines authorization, the test AC is orphaned. |
| R4 | Implementation leaking | LOW | Are requirements prescribing HOW (specific component names, file paths, architecture choices) instead of WHAT (observable behavior)? Implementation details belong in design.md. |
| R5 | Phase completeness | MEDIUM | If work is phased, is every phase transition state defined? Example: 5 sections exist in Phase 1 but section 4 content is Phase 2 -- what does section 4 show in Phase 1? |
| R6 | EARS compliance | LOW | Do ACs use "shall" language? Are they testable and verifiable? Do they describe single behaviors? |
| R7 | Test coverage requirements | HIGH | Is there a unit test requirement with coverage threshold? Is there an E2E test requirement? Are test scenarios concrete enough to implement? |

### Design Review

| # | Check | Severity | What to Look For |
|---|-------|----------|------------------|
| D1 | Architecture alignment | HIGH | Does the design integrate with existing patterns? Read current source files -- does the design account for the project's actual data fetching, state management, and error handling patterns? |
| D2 | Component interface consistency | HIGH | Do proposed interfaces match existing patterns in the codebase? Check prop patterns, hook signatures, error handling conventions. |
| D3 | Missing integration points | MEDIUM | Does the design cover how new components wire into existing ones? Layout imports, route registration, navigation updates. |
| D4 | Data flow completeness | MEDIUM | Is every data fetch described? Are loading, error, and empty states covered for each async operation? |
| D5 | Testability section | MEDIUM | Does the design include a testability section? Mock boundaries, test data strategy, component isolation approach. |
| D6 | Design review rules compliance | LOW | Cross-check against `.kiro/settings/rules/design-review.md` criteria if the file exists. |

### Tasks Review

| # | Check | Severity | What to Look For |
|---|-------|----------|------------------|
| T1 | Requirements traceability | HIGH | Every requirement must map to at least one task. Every task must reference requirement IDs. Cross-check both directions. |
| T2 | Task actionability | MEDIUM | Are tasks concrete enough to implement without guessing? Vague tasks like "set up section" without specifying what data to show are not actionable. |
| T3 | Testing task completeness | HIGH | Dedicated unit test task group exists. Dedicated E2E test task group exists. Coverage targets specified. Testing tasks are NOT marked optional. |
| T4 | Dependency ordering | MEDIUM | Do tasks reference outputs of earlier tasks? Is the execution order logical? Can a developer start task 1 without reading task 5 first? |
| T5 | Parallel analysis correctness | LOW | If `(P)` markers exist, verify the marked tasks truly have no data/file dependencies. If tasks share target files, they cannot be parallel. |
| T6 | Task generation rules compliance | LOW | Cross-check against `.kiro/settings/rules/tasks-generation.md` if the file exists. Natural language descriptions, 2-level hierarchy, sequential numbering. |

## Output Format

```
+-- Kiro Review: {artifact-type} -- {feature-name} ----------+
|                                                             |
|  Codebase explored: {N files read, key components found}    |
|                                                             |
|  Issues Found:                                              |
|                                                             |
|  [HIGH] Issue title                                         |
|     Evidence: {what you found in code/spec}                 |
|     Impact: {what goes wrong if not fixed}                  |
|     Fix: {specific revision guidance}                       |
|                                                             |
|  [MED]  Issue title                                         |
|     Evidence: ...                                           |
|     Impact: ...                                             |
|     Fix: ...                                                |
|                                                             |
|  [LOW]  Issue title                                         |
|     Evidence: ...                                           |
|     Fix: ...                                                |
|                                                             |
|  Strengths:                                                 |
|  - {1-2 things done well}                                   |
|                                                             |
|  -----------------------------------------------------------+
|  Verdict: GO / NO-GO                                        |
|  Reason: {1-2 sentences}                                    |
|  Action: {what to do next}                                  |
+-------------------------------------------------------------+
```

## Verdict Rules

- **GO**: Zero HIGH issues and zero unfixed MED issues.
- **NO-GO**: One or more HIGH issues remain after fixes.
- **GO with conditions**: Zero HIGH but deferred MED issues exist that the user should acknowledge.
- **NEEDS-INPUT**: The reviewer found an issue meeting the escalation threshold (see Autonomy Boundaries). Returns the specific question with valid options and their trade-offs. The pipeline pauses for user input, then the reviewer re-runs with the answer.

## Autonomy Boundaries

The reviewer operates autonomously by default. It uses a ternary decision model for every issue found:

### Decision Model

| Action | Criteria | Example |
|--------|----------|---------|
| **Fix** | Gap in the current artifact that has one clear correct resolution. The fix does not change product scope or user-facing behavior -- it fills in what was obviously missing. | Missing E2E scenario for a section that exists in requirements. Orphaned test AC referencing undefined behavior. |
| **Defer** | Concern that belongs to a later phase. The current artifact cannot resolve it because the answer depends on design, implementation, or data that doesn't exist yet. | Requirements reference a data source that doesn't exist -- design will define the data model. |
| **Escalate** | Two or more valid approaches exist that represent **different product decisions**, and no evidence in the spec, codebase, or prior decisions resolves the ambiguity. The reviewer cannot pick one without making a product judgment call. | A section could show summary metrics OR a detail table -- both are valid UX choices. A requirement could scope to Phase 1 or Phase 2 -- both are valid timeline choices. |

### Escalation Threshold

Escalate ONLY when ALL of these are true:
1. The issue affects **user-facing behavior or product scope** (not just technical approach)
2. There are **2+ valid options** with meaningfully different outcomes
3. No evidence in the spec, codebase, or pipeline-log resolves the ambiguity
4. The reviewer's choice would **commit the project to a direction** the user has not approved

If any condition is false, do NOT escalate -- fix or defer instead.

### Default Bias

When uncertain whether to fix, defer, or escalate:
- **Fix > Defer**: If you can fix it cleanly, fix it. Don't defer to avoid responsibility.
- **Defer > Escalate**: If the answer will naturally emerge in a later phase, defer. Don't escalate prematurely.
- **Escalate only as last resort**: Escalation interrupts the pipeline. Use it sparingly.

## Codebase Exploration Strategy

The reviewer MUST read actual source files, not just the spec. Strategy by artifact type:

**For requirements review:**
1. Identify the target page/component from the project description
2. Read it and catalog: imports, data fetches, rendered sections, existing components
3. Search `src/` (or equivalent) for related existing code -- components, hooks, utilities
4. Check for existing database schemas or migrations referenced in the description
5. Compare what the codebase has against what the requirements specify

**For design review:**
1. Read all source files the design proposes to modify or create
2. Check existing patterns (data fetching, state management, error handling)
3. Verify proposed interfaces against actual types in the codebase
4. Check route registration and layout integration

**For tasks review:**
1. Verify design.md component descriptions match actual file inventory
2. Check that task file targets don't conflict (parallel analysis validation)
3. Verify referenced hooks/utilities exist or are created by earlier tasks

## Integration with kiro:pipeline

The reviewer is invoked by `/kiro:pipeline` at these points:

| Pipeline Phase | Reviewer Invocation | Purpose |
|----------------|--------------------|---------|
| Phase 3: After gap analysis | `/kiro:reviewer {feature} requirements` | Substantive review before proceeding to design |
| Phase 5: After tasks generation | `/kiro:reviewer {feature} tasks` | Verify traceability and actionability before implementation |

The reviewer runs BEFORE presenting the artifact to the user for approval. Its verdict and issues are included in the presentation so the user can make an informed approval decision.

## Anti-Patterns

| Do NOT | Do Instead |
|--------|-----------|
| Review only the spec artifact in isolation | Read the codebase -- that's the whole point |
| Accept thin/hollow sections as "intentionally minimal" | Flag them as HIGH issues -- thin sections create poor outcomes |
| Assume referenced components exist | Verify by reading source files |
| Produce vague issues ("could be better") | Every issue needs Evidence, Impact, and Fix |
| Review implementation details in requirements | Flag them as LOW issues -- they belong in design |
| Escalate fixable gaps | Fix them. Escalation is a last resort. |
