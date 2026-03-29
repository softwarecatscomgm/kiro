---
description: Execute spec tasks using TDD methodology
allowed-tools: Read, Task
argument-hint: <feature-name> [task-numbers]
---

# Implementation Task Executor

## Parse Arguments
- Feature name: `$0`
- Task numbers: `$1` (optional)
  - Format: "1.1" (single task) or "1,2,3" (multiple tasks)
  - If not provided: Execute all pending tasks

## Validate
Check that tasks have been generated:
- Verify `.kiro/specs/$0/` exists
- Verify `.kiro/specs/$0/tasks.md` exists

If validation fails, inform user to complete tasks generation first.

## Task Selection Logic

**Parse task numbers from `$1`** (perform this in Slash Command before invoking Subagent):
- If `$1` provided: Parse task numbers (e.g., "1.1", "1,2,3")
- Otherwise: Read `.kiro/specs/$0/tasks.md` and find all unchecked tasks (`- [ ]`)

## Invoke Subagent

Delegate TDD implementation to spec-tdd-impl-agent:

Use the Task tool to invoke the Subagent with file path patterns:

```
Task(
  subagent_type="spec-tdd-impl-agent",
  description="Execute TDD implementation",
  prompt="""
Feature: $0
Spec directory: .kiro/specs/$0/
Target tasks: {parsed task numbers or "all pending"}

File patterns to read:
- .kiro/specs/$0/*.{json,md}
- .kiro/steering/*.md

TDD Mode: strict (test-first)
"""
)
```

## Post-Execution: Check Completion

After the subagent returns, read `.kiro/specs/$0/tasks.md` and count unchecked tasks (`- [ ]`):

- **If zero unchecked tasks remain**: All tasks are complete. Update `.kiro/specs/$0/spec.json`:
  - Set `"phase"` to `"implementation-complete"`
  - Set `"updated_at"` to the current ISO 8601 timestamp
  - Inform the user that all tasks are done and the spec phase has been updated

- **If unchecked tasks remain**: Report how many tasks are pending and list them.

## Display Result

Show Subagent summary to user, then provide next step guidance:

### Task Execution

**Execute specific task(s)**:
- `/kiro:spec-impl $0 1.1` - Single task
- `/kiro:spec-impl $0 1,2,3` - Multiple tasks

**Execute all pending**:
- `/kiro:spec-impl $0` - All unchecked tasks

**Before Starting Implementation**:
- **IMPORTANT**: Clear conversation history and free up context before running `/kiro:spec-impl`
- This applies when starting first task OR switching between tasks
- Fresh context ensures clean state and proper task focus
