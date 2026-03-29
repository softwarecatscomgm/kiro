---
description: Interactive technical design quality review and validation
allowed-tools: Read, Task
argument-hint: <feature-name>
---

# Technical Design Validation

## Parse Arguments
- Feature name: `$0`

## Validate
Check that design has been completed:
- Verify `.kiro/specs/$0/` exists
- Verify `.kiro/specs/$0/design.md` exists

If validation fails, inform user to complete design phase first.

## Invoke Subagent

Delegate design validation to validate-design-agent:

Use the Task tool to invoke the Subagent with file path patterns:

```
Task(
  subagent_type="validate-design-agent",
  description="Interactive design review",
  prompt="""
Feature: $0
Spec directory: .kiro/specs/$0/

File patterns to read:
- .kiro/specs/$0/spec.json
- .kiro/specs/$0/requirements.md
- .kiro/specs/$0/design.md
- .kiro/steering/*.md
- .kiro/settings/rules/design-review.md
"""
)
```

## Display Result

Show Subagent summary to user, then provide next step guidance:

### Next Phase: Task Generation

**If Design Passes Validation (GO Decision)**:
- Review feedback and apply changes if needed
- Run `/kiro:spec-tasks $0` to generate implementation tasks
- Or `/kiro:spec-tasks $0 -y` to auto-approve and proceed directly

**If Design Needs Revision (NO-GO Decision)**:
- Address critical issues identified
- Re-run `/kiro:spec-design $0` with improvements
- Re-validate with `/kiro:validate-design $0`

**Note**: Design validation is recommended but optional. Quality review helps catch issues early.
