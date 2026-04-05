---
name: pipeline-orchestration
description: Use when running a feature through the full kiro spec-driven lifecycle. Orchestrates init, requirements, design, tasks, and implementation with quality gates, audit trail, and reviewer integration.
license: MIT
compatibility: Claude Code, Cursor, VS Code, Windsurf
metadata:
  category: methodology
  complexity: advanced
  author: Kiro Team
  version: "1.0.0"
---

# Pipeline Orchestration

Orchestrate the full kiro spec-driven lifecycle for a feature -- from description to validated code -- with quality gates, an audit trail, and automatic implementation mode selection.

## Invocation

```
/kiro:pipeline "feature description..."                        # New feature
/kiro:pipeline {feature-name}                                  # Resume existing spec
/kiro:pipeline {feature-name} --from=design                    # Resume from specific phase
/kiro:pipeline {feature-name} --sequential                     # Force sequential implementation
```

## Phase Detection

On invocation, determine the starting point:

1. **If argument is a quoted string**: New feature. Start at Phase 1 (Init).
2. **If argument matches a directory in `.kiro/specs/`**: Existing spec. Read `spec.json` to determine current phase.
3. **If `--from=` flag present**: Override auto-detection, start at specified phase.
4. **If `pipeline-log.md` exists**: Read it to understand prior decisions.

### Phase-to-Resume Mapping

| spec.json `phase` | Resume from |
|--------------------|-------------|
| `initialized` | Phase 2: Requirements |
| `requirements` | Phase 3: Gap Analysis or Phase 4: Design |
| `design` | Phase 5: Tasks |
| `tasks-generated` | Phase 6: Implementation |
| `implementation-complete` | Phase 7: Validation |
| `completed` | Nothing -- inform user spec is done |

## Pipeline Phases

### Phase 1: Init

**Skip if**: spec already exists.

1. Invoke `/kiro:spec-init` with the feature description
2. Log to `pipeline-log.md`:
   - Feature name, timestamp, description

### Phase 2: Requirements

1. Invoke `/kiro:spec-requirements {feature}`
2. **Quality Gate -- Mechanical Checks**:
   - Read generated `requirements.md`
   - Verify at least one requirement covers unit test coverage (threshold from project config, default 80%+)
   - Verify at least one requirement covers E2E test scenarios
   - Verify requirements use EARS format with numeric IDs
   - If missing: report which test requirements are absent, re-invoke with guidance
3. Log: requirement count, test requirement IDs, reviewer verdict, issue count by severity, deferred issues
4. Proceed to next phase automatically (no manual approval gate)

### Phase 3: Gap Analysis (conditional)

Run gap analysis.

1. Invoke `/kiro:validate-gap {feature}`
2. Log: existing components found, integration points, strategy
3. **Quality Gate -- Substantive Review**:
   - Invoke `/kiro:reviewer {feature} requirements`
   - The reviewer runs as an **autonomous subagent**: it explores the codebase, applies the review checklist, fixes fixable gaps directly in requirements.md, and returns a final verdict
   - If NO-GO (HIGH issues the reviewer could not fix): log issues, present to user, re-invoke requirements generation with feedback
   - If GO or GO-with-conditions: the reviewer has already fixed fixable issues; only deferred concerns remain. Proceed automatically.
   - If NEEDS-INPUT: present the reviewer's question to the user, get answer, re-invoke reviewer with answer

### Phase 4: Design

1. Invoke `/kiro:spec-design {feature}`
2. **Quality Gate -- Structural Review**:
   - Invoke `/kiro:validate-design {feature}`
   - Require GO decision
   - Verify testability section is present in `design.md`
   - Verify data isolation strategy is addressed
   - If NO-GO: log critical issues, present to user, re-invoke design with feedback
3. Log: validate-design verdict, reviewer verdict, combined critical issue count, testability check, deferred issues
4. Proceed to next phase automatically (no manual approval gate)

### Phase 5: Tasks

1. Invoke `/kiro:spec-tasks {feature}`
2. **Quality Gate -- Mechanical Checks**:
   - Read generated `tasks.md`
   - Verify dedicated unit testing task group exists
   - Verify dedicated E2E testing task group exists
   - Verify coverage targets specified (project default or 80% statements)
   - Verify all requirements traced to at least one task (`_Requirements: X.X_`)
   - Verify testing tasks are NOT marked optional (`- [ ]*`)
   - If missing: report gaps, re-invoke with corrections
3. **Quality Gate -- Substantive Review**:
   - Invoke `/kiro:reviewer {feature} tasks`
   - The reviewer runs as an **autonomous subagent**: it verifies requirements traceability in both directions, checks task actionability, validates parallel analysis against actual file targets, fixes fixable gaps directly in tasks.md, and returns a final verdict
   - If NO-GO (HIGH issues the reviewer could not fix): log issues, present to user, re-invoke tasks generation with feedback
   - If GO or GO-with-conditions: the reviewer has already fixed fixable issues; only deferred concerns remain. Proceed automatically.
   - If NEEDS-INPUT: present the reviewer's question to the user, get answer, re-invoke reviewer with answer
4. **Parallel Stream Analysis**: Count `(P)` markers to determine implementation mode
5. Log: task count, testing task IDs, reviewer verdict, parallel stream count, coverage target, deferred issues
6. Proceed to next phase automatically (no manual approval gate)

### Phase 6: Implementation

1. Invoke `/kiro:spec-impl {feature}` (all pending tasks)
2. If `--sequential` was set, note in log
3. Log: mode, task execution order

**Post-implementation completeness check:**
After execution finishes, read `tasks.md` and verify ALL tasks are `[x]`. Do NOT log Phase 6 as COMPLETE if any tasks remain `[ ]`. Either execute the remaining tasks or explicitly log them as deferred with a reason. A Phase 6 status of `COMPLETE` means every task in tasks.md is checked.

### Phase 7: Validation

**Step 1 -- Tasks completeness gate (mandatory):**
Read `tasks.md` and verify ALL tasks are `[x]`. If any tasks remain `[ ]`, Phase 7 verdict is **NO-GO**. List the unchecked tasks in the log. Do NOT proceed to verification commands until every task is checked.

**Step 2 -- Automated verification:**
1. Invoke `/kiro:validate-impl {feature}`
2. Run the project's verification commands:
   - Build command (e.g., `npm run build`, `make build`, `cargo build`) -- compilation/transpilation
   - Unit test command (e.g., `npm run test`, `pytest`, `go test`) -- unit tests pass
   - Coverage command (if available) -- coverage thresholds met

Discover project commands from: steering docs (`.kiro/steering/tech.md`), `package.json` scripts, `Makefile`/`justfile` targets, or `README.md`. If unclear, ask the user.

**Step 3 -- E2E test gate (mandatory when E2E tasks exist):**
If `tasks.md` contains E2E test tasks, E2E tests MUST run before the pipeline can declare GO. Ask the user for permission to run E2E tests. If the user declines, the Phase 7 verdict is **GO-PENDING-E2E** (not GO). `spec.json` phase stays `implementation-complete` (not `completed`). The pipeline log marks `Status: COMPLETE (pending E2E)`. The feature is NOT done until E2E tests pass.

**Step 4 -- Verdict:**
- **GO**: All tasks `[x]`, build passes, unit tests pass, coverage met, E2E tests pass
- **GO-PENDING-E2E**: All automated checks pass but E2E tests were not run (user declined or deferred)
- **NO-GO**: Any check fails -- log issues, present remediation plan

3. Log: validation result, test pass/fail, coverage numbers, E2E status

## Audit Trail

All pipeline activity is logged to `.kiro/specs/{feature}/pipeline-log.md`.

**Rules:**
- Append-only -- never modify previous entries
- Every phase gets a timestamped section header
- Log decisions, approvals, quality gate results, and key metrics
- Log failures and re-invocations with reasons

**Format:**

```markdown
# Pipeline Log: {feature-name}

## Phase N: {name} -- {ISO-8601 timestamp}
- Key metric 1: value
- Key metric 2: value
- Quality gate: PASS/FAIL (reason)
- User approval: APPROVED/REJECTED (feedback if rejected)
- Status: COMPLETE/FAILED/SKIPPED
```

## Quality Standards Reference

These thresholds are enforced at quality gates:

| Standard | Threshold | Enforced At |
|----------|-----------|-------------|
| Unit test coverage | Project default (80%+ recommended) | Phase 2 (requirement), Phase 7 (validation) |
| E2E test coverage | Project default (80%+ recommended) | Phase 2 (requirement), Phase 7 (validation) |
| Substantive requirements review | GO from `/kiro:reviewer` | Phase 3 |
| Design review (structural) | GO from `/kiro:validate-design` | Phase 4 |
| Requirements traceability | All reqs mapped to tasks | Phase 5 |
| Tasks review (substantive) | GO from `/kiro:reviewer` | Phase 5 |
| Testing tasks | Dedicated groups, not optional | Phase 5 |
| All tasks checked | Every `[ ]` in tasks.md must be `[x]` | Phase 6 (post-impl), Phase 7 (pre-validation) |
| Build passes | Project build command succeeds | Phase 7 |
| E2E tests pass | All E2E scenarios green (when E2E tasks exist) | Phase 7 (mandatory gate) |

## Error Recovery

| Failure | Recovery |
|---------|----------|
| Mechanical quality gate fails | Log failure reason, present to user, re-invoke phase with guidance |
| `/kiro:reviewer` returns NO-GO | Log HIGH issues with evidence, revise artifact to address them, re-invoke reviewer |
| `/kiro:reviewer` returns GO-with-conditions | Log deferred issues in pipeline-log.md. Proceed automatically. Do NOT pause for user acknowledgment -- deferred issues are informational, not blocking. |
| `/kiro:reviewer` returns NEEDS-INPUT | Present the reviewer's specific question and options to the user. Wait for answer. Re-invoke reviewer with the answer. |
| Design gets NO-GO from validate-design | Log critical issues, re-invoke `/kiro:spec-design` with review feedback |
| Implementation fails | Log error, identify failing task, re-invoke for that specific task |
| Tests fail in validation | Log failures, create remediation tasks, re-invoke implementation |
| User rejects phase output (when asked via NEEDS-INPUT) | Log rejection reason, re-invoke phase |

## Anti-Patterns

| Do NOT | Do Instead |
|--------|-----------|
| Pause the pipeline for manual approval when reviewer gave GO | Proceed automatically -- only NEEDS-INPUT and NO-GO pause the pipeline |
| Continue after NO-GO design review | Fix design issues before proceeding to tasks |
| Mark testing tasks as optional (`*`) | Testing is mandatory -- first-class tasks |
| Run implementation after NO-GO verdict | Fix issues before proceeding -- but GO/GO-with-conditions proceed automatically |
| Modify pipeline-log.md entries retroactively | Append new entries -- the log is immutable |
| Run E2E tests without permission | Ask the user before running E2E tests |
| Escalate fixable gaps to user as "conditions" | Fix them in the artifact, re-review, then present clean results |
| Declare Phase 6 COMPLETE with unchecked tasks | Verify every `[ ]` is `[x]` before marking COMPLETE |
| Skip E2E gate in Phase 7 because user hasn't given permission yet | Ask for permission. If declined, use GO-PENDING-E2E verdict -- never plain GO |
| Set spec.json phase to `completed` without E2E passing | Phase stays `implementation-complete` until E2E tests run and pass |
| Present a phase summary table with "next steps" options after any phase that has no manual approval gate | Invoke the next phase immediately. Summary output belongs in pipeline-log.md, not as a user prompt. A summary followed by "run X to proceed" IS a manual gate — it has the same effect as pausing for approval. |
