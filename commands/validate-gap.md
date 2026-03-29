---
description: Analyze implementation gap between requirements and existing codebase
allowed-tools: Read, Task
argument-hint: <feature-name>
---

# Implementation Gap Validation

## Parse Arguments
- Feature name: `$0`

## Validate
Check that requirements have been completed:
- Verify `.kiro/specs/$0/` exists
- Verify `.kiro/specs/$0/requirements.md` exists

If validation fails, inform user to complete requirements phase first.

## Invoke Subagent

Delegate gap analysis to validate-gap-agent:

Use the Task tool to invoke the Subagent with file path patterns:

```
Task(
  subagent_type="validate-gap-agent",
  description="Analyze implementation gap",
  prompt="""
Feature: $0
Spec directory: .kiro/specs/$0/

File patterns to read:
- .kiro/specs/$0/spec.json
- .kiro/specs/$0/requirements.md
- .kiro/steering/*.md
- .kiro/settings/rules/gap-analysis.md
"""
)
```

## Display Result

Show Subagent summary to user, then provide next step guidance:

### Next Phase: Design Generation

**If Gap Analysis Complete**:
- Review gap analysis insights
- Run `/kiro:spec-design $0` to create technical design document
- Or `/kiro:spec-design $0 -y` to auto-approve requirements and proceed directly

**Note**: Gap analysis is optional but recommended for brownfield projects to inform design decisions.
