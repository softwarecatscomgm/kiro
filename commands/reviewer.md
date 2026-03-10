---
description: Critical review of kiro spec artifacts against codebase and quality standards
allowed-tools: Read, Task, Glob
argument-hint: feature-name [requirements|design|tasks]
---

# Kiro Reviewer

Autonomous substantive review of a kiro spec artifact. Runs as a subagent that explores the codebase, identifies issues, fixes fixable gaps, and returns a final verdict.

## Variables

INPUT: $ARGUMENTS

## Pre-flight (runs in main thread)

1. Parse INPUT:
   - **First argument**: feature name (must match a directory in `.kiro/specs/`)
   - **Second argument**: artifact type -- one of `requirements`, `design`, `tasks`
   - If second argument missing, auto-detect from spec.json phase:
     - `requirements-generated` or `requirements` -> review requirements
     - `design` -> review design
     - `tasks-generated` -> review tasks
2. Verify the target artifact file exists:
   - `requirements` -> `.kiro/specs/{feature}/requirements.md`
   - `design` -> `.kiro/specs/{feature}/design.md`
   - `tasks` -> `.kiro/specs/{feature}/tasks.md`

## Execute (delegated to autonomous subagent)

Invoke the **entire review** as a single subagent using the Task tool:

```
Task(
  subagent_type="general-purpose",
  description="Review {artifact-type} for {feature}",
  prompt="""
You are the kiro-reviewer -- an autonomous code reviewer for spec artifacts.

## Your Task
Perform a substantive critical review of `.kiro/specs/{feature}/{artifact}.md` by cross-referencing against the actual codebase.

## Review Workflow
1. Read: `.kiro/specs/{feature}/spec.json`, `.kiro/specs/{feature}/{artifact}.md`, `.kiro/specs/{feature}/pipeline-log.md` (if exists)
2. Explore the codebase: Read target source files referenced in the artifact. Identify what EXISTS vs what the artifact ASSUMES exists. Read imports, data fetches, rendered sections, hook signatures.
3. Apply the review checklist from the kiro:spec-reviewer skill (load the skill for the full checklist).
4. Classify each issue found:
   - **Fixable**: A gap or deficiency in the current artifact that you can fix (missing ACs, incomplete test scenarios, orphaned references). FIX THESE DIRECTLY in the artifact file.
   - **Deferred**: A concern that belongs in the next phase (data sourcing questions for design, architecture decisions for implementation). NOTE THESE but do not fix.
5. If you fixed any issues, re-read the artifact to verify your fixes are clean.
6. Produce the final verdict using the output format from the skill.

## Verdict Rules
- **GO**: Zero HIGH issues and zero unfixed MED issues.
- **NO-GO**: One or more HIGH issues remain after your fixes.
- **GO with conditions**: Zero HIGH but deferred MED issues exist that the user should acknowledge.

## Autonomy Rules
- Fix all fixable issues WITHOUT asking. You have write access to the artifact file.
- Do NOT fix source code files -- only spec artifacts (.kiro/specs/ files).
- Defer next-phase concerns WITHOUT asking. Document them in the verdict.
- Escalate ONLY when the autonomy boundary criteria in the kiro:spec-reviewer skill are met (product scope decision, 2+ valid options, no resolving evidence). Return a NEEDS-INPUT verdict with the specific question and options.
- Default bias: Fix > Defer > Escalate. When uncertain, do NOT escalate.

## Output
Return your verdict in the format specified by the kiro:spec-reviewer skill, plus a summary of any fixes you applied.
"""
)
```

## Report

The subagent returns the final verdict directly. Present it to the user as-is:
- If GO: report the verdict and any fixes applied
- If GO-with-conditions: report the verdict with deferred issues noted
- If NO-GO: report the HIGH issues that could not be autonomously resolved
- If NEEDS-INPUT: present the reviewer's question with options to the user
