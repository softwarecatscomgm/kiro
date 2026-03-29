---
description: Generate comprehensive requirements for a specification
allowed-tools: Read, Task
argument-hint: <feature-name>
---

# Requirements Generation

## Parse Arguments
- Feature name: `$0`

## Validate
Check that spec has been initialized:
- Verify `.kiro/specs/$0/` exists
- Verify `.kiro/specs/$0/spec.json` exists

If validation fails, inform user to run `/kiro:spec-init` first.

## Invoke Subagent

Delegate requirements generation to spec-requirements-agent:

Use the Task tool to invoke the Subagent with file path patterns:

```
Task(
  subagent_type="spec-requirements-agent",
  description="Generate EARS requirements",
  prompt="""
Feature: $0
Spec directory: .kiro/specs/$0/

File patterns to read:
- .kiro/specs/$0/spec.json
- .kiro/specs/$0/requirements.md
- .kiro/steering/*.md
- .kiro/settings/rules/ears-format.md
- .kiro/settings/templates/specs/requirements.md

Mode: generate
"""
)
```

## Display Result

Show Subagent summary to user, then provide next step guidance:

### Next Phase: Design Generation

**If Requirements Approved**:
- Review generated requirements at `.kiro/specs/$0/requirements.md`
- **Optional Gap Analysis** (for existing codebases):
  - Run `/kiro:validate-gap $0` to analyze implementation gap with current code
  - Identifies existing components, integration points, and implementation strategy
  - Recommended for brownfield projects; skip for greenfield
- Then `/kiro:spec-design $0 [-y]` to proceed to design phase

**If Modifications Needed**:
- Provide feedback and re-run `/kiro:spec-requirements $0`

**Note**: Approval is mandatory before proceeding to design phase.
