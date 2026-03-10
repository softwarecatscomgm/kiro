# Kiro Plugin for Claude Code

A Claude Code plugin providing spec-driven development with agents, commands, and skills.

## Installation

### From Marketplace (Recommended)

```bash
# In Claude Code
/plugin marketplace add softwarecatscomgm/scc-marketplace
/plugin install kiro@scc-marketplace
```

### Installation Scopes

```bash
# User scope (default) - available in all projects
/plugin install kiro@scc-marketplace

# Project scope - shared with team via git
/plugin install kiro@scc-marketplace --scope project

# Local scope - project-specific, gitignored
/plugin install kiro@scc-marketplace --scope local
```

## Included Skills

Once installed, Claude will automatically use these skills when relevant:

| Skill | Description | Triggers |
|-------|-------------|----------|
| **spec-driven-development** | Complete three-phase methodology | "create a spec", "plan this feature", "spec-driven" |
| **requirements-engineering** | EARS format requirements | "requirements", "acceptance criteria", "user stories" |
| **design-documentation** | Technical architecture | "design document", "architecture", "technical design" |
| **task-breakdown** | Implementation planning | "break down tasks", "implementation plan", "task list" |
| **pipeline-orchestration** | Full lifecycle with quality gates | "run the full pipeline", "end-to-end spec" |
| **spec-reviewer** | Autonomous spec artifact review | "review this spec", "quality gate", "GO/NO-GO" |
| **ai-prompting** | AI communication strategies | "prompt better", "AI communication", "improve prompts" |
| **quality-assurance** | Testing and validation | "quality", "testing strategy", "validation" |
| **troubleshooting** | Problem resolution | "debug", "troubleshoot", "issue with" |
| **create-steering-documents** | Project guidelines setup | "steering documents", "project standards", "setup guidelines" |

## Available Commands

| Command | Description |
|---------|-------------|
| `/kiro:spec-init` | Initialize a new specification |
| `/kiro:spec-requirements` | Generate requirements |
| `/kiro:spec-design` | Create technical design |
| `/kiro:spec-tasks` | Generate implementation tasks |
| `/kiro:spec-impl` | Execute tasks with TDD |
| `/kiro:spec-quick` | Quick spec generation |
| `/kiro:spec-status` | Show spec progress |
| `/kiro:pipeline` | Full lifecycle with quality gates and audit trail |
| `/kiro:reviewer` | Autonomous spec artifact review against codebase |
| `/kiro:steering` | Manage steering documents |
| `/kiro:steering-custom` | Create custom steering documents |
| `/kiro:validate-design` | Validate technical design |
| `/kiro:validate-gap` | Analyze implementation gaps |
| `/kiro:validate-impl` | Validate implementation |

## Verifying Installation

```bash
# List installed plugins
/plugin

# Check plugin status
/plugin info kiro
```

## Updating

```bash
/plugin update kiro@scc-marketplace
```

## Uninstalling

```bash
/plugin uninstall kiro
```

## Plugin Contents

```
kiro/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest
├── agents/                    # 9 agents
├── commands/                  # 14 commands
├── skills/                    # 10 skills
├── mcp-server/                # Optional MCP server
└── spec-process-guide/        # Full documentation
```

## License

MIT - See [LICENSE](LICENSE)
