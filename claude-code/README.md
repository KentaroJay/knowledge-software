# Claude Code Documentation

A comprehensive guide to Claude Code CLI - Anthropic's official command-line tool for working with Claude in your codebase.

## Contents

1. [CLAUDE.md & Project Memory](./claude-md-project-memory.md) - How Claude remembers project context
2. [Hooks System](./hooks-system.md) - Automate actions at lifecycle events
3. [Custom Skills (Slash Commands)](./custom-skills.md) - Create reusable workflows with `/commands`
4. [Custom Agents (Subagents)](./custom-agents.md) - Specialized AI assistants for delegated tasks
5. [MCP Servers](./mcp-servers.md) - Connect Claude to external tools and services
6. [Settings & Permissions](./settings-and-permissions.md) - Configure access control and preferences
7. [Tips & Workflows](./tips-and-workflows.md) - Keyboard shortcuts, modes, and productivity tips

## Quick Reference

### Key Files

| File | Purpose | Shared via Git? |
|------|---------|-----------------|
| `CLAUDE.md` | Project memory & instructions | Yes |
| `.claude/settings.json` | Team settings & hooks | Yes |
| `.claude/settings.local.json` | Personal settings | No |
| `.claude/agents/*.md` | Custom subagents | Yes |
| `.claude/skills/*/SKILL.md` | Custom slash commands | Yes |
| `.claude/rules/*.md` | Modular rule files | Yes |
| `.mcp.json` | MCP server config | Yes |

### Essential Commands

```bash
claude                    # Start interactive session
claude -p "prompt"        # One-off query (headless)
claude --continue         # Resume last session
claude --resume           # Pick a session to resume
/help                     # In-session help
/compact                  # Compress conversation context
/model                    # Switch AI model
/mcp                      # Manage MCP servers
```

## Resources

- [Official Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub Repository](https://github.com/anthropics/claude-code)
- [Issue Tracker](https://github.com/anthropics/claude-code/issues)
