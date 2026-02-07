# Tips & Workflows

Keyboard shortcuts, modes, and productivity tips for Claude Code.

## Quick Summary

- Claude Code has extensive keyboard shortcuts for fast navigation
- Multiple modes: normal, plan, vim, and different permission levels
- Session management lets you pause, resume, and branch conversations
- Context management prevents running out of conversation space
- Headless mode enables scripting and automation

## Context

Knowing the shortcuts and workflows turns Claude Code from "a chatbot in your terminal" into a power tool. These tips help you work faster and avoid common frustrations.

## Keyboard Shortcuts

### Essential Shortcuts

| Shortcut | Action |
|----------|--------|
| `Enter` | Submit message |
| `Shift+Enter` / `Option+Enter` | New line (without submitting) |
| `Ctrl+C` | Cancel current generation |
| `Ctrl+D` | Exit Claude Code |
| `Esc` + `Esc` | Rewind conversation (undo last exchange) |
| `Shift+Tab` | Cycle permission modes |

### Navigation & Editing

| Shortcut | Action |
|----------|--------|
| `Ctrl+L` | Clear terminal screen |
| `Ctrl+R` | Reverse search history |
| `Ctrl+G` | Open input in external editor |
| `Ctrl+K` | Delete to end of line |
| `Ctrl+U` | Delete entire line |
| `Ctrl+Y` | Paste deleted text |

### Claude-Specific

| Shortcut | Action |
|----------|--------|
| `Ctrl+V` / `Cmd+V` | Paste image from clipboard |
| `Ctrl+O` | Toggle verbose output |
| `Ctrl+T` | Toggle task list |
| `Option+P` / `Alt+P` | Switch model |
| `Option+T` / `Alt+T` | Toggle extended thinking |
| `Tab` | Accept autocomplete suggestion |

### Customizing Keybindings

Edit `~/.claude/keybindings.json`:

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+shift+d": "chat:submit"
      }
    }
  ]
}
```

## Modes

### Permission Modes (Shift+Tab to cycle)

```
┌─────────────┐   Shift+Tab   ┌───────────────┐
│   Default    │ ────────────▶ │ Accept Edits  │
│  (ask most)  │               │ (auto-edits)  │
└─────────────┘               └───────┬───────┘
       ▲                              │
       │                        Shift+Tab
  Shift+Tab                           │
       │                              ▼
┌──────┴──────┐               ┌───────────────┐
│    Plan      │ ◀──────────── │  Don't Ask    │
│ (read-only)  │   Shift+Tab  │ (auto-approve)│
└─────────────┘               └───────────────┘
```

| Mode | Best For |
|------|----------|
| **Default** | Normal development - asks before most actions |
| **Accept Edits** | Rapid prototyping - auto-approves file changes |
| **Don't Ask** | Trusted tasks - auto-approves almost everything |
| **Plan** | Research and planning - no file modifications |

### Plan Mode

Enter with `/plan` or `Shift+Tab` to Plan. Claude can:
- Read and explore the codebase
- Design an implementation approach
- Present a plan for your approval
- Cannot modify any files

### Vim Mode

Toggle with `/vim`. Supports:
- **NORMAL mode** - `h/j/k/l` navigation, `d/c/y` operators
- **INSERT mode** - Standard text editing
- **Text objects** - `iw`, `aw`, `i"`, `i(`, etc.

## Session Management

### Resuming Sessions

```bash
claude --continue              # Resume most recent session
claude --resume                # Pick from recent sessions
```

### Inside Claude Code

```
/rename my-feature-work        # Label a session
/export                        # Export conversation
/clear                         # Start fresh
```

### Rewind

Press `Esc` + `Esc` to undo the last exchange. Useful when Claude goes down the wrong path.

## Context Management

Claude Code has a limited conversation context window. Manage it:

### Check Usage

```
/context                       # Visualize context usage
/cost                          # Show token usage
```

### Compress Context

```
/compact                       # Summarize conversation to free space
/compact "focus on the auth refactor"  # Custom summary focus
```

Auto-compaction happens at ~95% capacity.

### Tips for Context Efficiency

- Use subagents (`Task` tool) for research-heavy work - isolates context
- Use `/compact` before starting a new sub-task in the same session
- Break large tasks into separate sessions
- Use `Esc+Esc` to rewind if Claude went off track (saves context vs. correcting)

## Headless Mode (Scripting)

Run Claude Code non-interactively:

```bash
# One-off query
claude -p "explain this error: $(cat error.log)"

# JSON output
claude -p "list all TODO comments" --output-format json

# Streaming JSON
claude -p "analyze codebase" --output-format stream-json

# Pipe input
cat file.py | claude -p "review this code"
```

### Use Cases

- CI/CD pipelines
- Pre-commit hooks
- Automated code review
- Documentation generation

## Useful Built-in Commands

| Command | Purpose |
|---------|---------|
| `/help` | Get help |
| `/compact` | Compress conversation |
| `/context` | Show context usage |
| `/cost` | Show token usage |
| `/model` | Switch AI model |
| `/mcp` | Manage MCP servers |
| `/memory` | Edit CLAUDE.md files |
| `/init` | Initialize CLAUDE.md |
| `/permissions` | Manage permissions |
| `/plan` | Enter plan mode |
| `/resume` | Resume a session |
| `/rewind` | Undo last exchange |
| `/theme` | Change color theme |
| `/vim` | Toggle vim mode |
| `/debug` | Debug current session |
| `/doctor` | Check installation health |

## Productivity Workflows

### 1. Research-Then-Implement

```
1. /plan                          # Enter plan mode
2. Ask Claude to explore and plan
3. Review the plan
4. Shift+Tab to Default mode
5. "Implement the plan"
```

### 2. Multi-File Refactor

```
1. Describe the refactor goal
2. Let Claude create a plan with task list
3. Ctrl+T to monitor progress
4. Review changes with git diff
```

### 3. Learning a New Codebase

```
1. /plan                          # Safe read-only mode
2. "Explain the architecture of this project"
3. "How does authentication work?"
4. "What would I need to change to add feature X?"
```

### 4. Debugging

```
1. Paste error message or screenshot (Ctrl+V for images)
2. "Find the root cause"
3. Let Claude explore and diagnose
4. Review the proposed fix before approving
```

## Key Takeaways

- Learn `Shift+Tab` (permission modes) and `Esc+Esc` (rewind) - they save the most time
- Use Plan mode for exploration and research before committing to changes
- `/compact` is your friend for long sessions
- Headless mode (`-p`) enables powerful scripting and automation
- Paste images with `Ctrl+V` for visual debugging
- Use `--continue` and `--resume` to pick up where you left off

## Common Pitfalls

- **Running out of context** - Use `/compact` proactively, don't wait for auto-compaction
- **Wrong permission mode** - Check the mode indicator; Don't Ask mode approves everything
- **Not using rewind** - Esc+Esc is cheaper than explaining "no, go back" to Claude
- **Ignoring plan mode** - For complex tasks, planning first prevents wasted effort
- **Not labeling sessions** - Use `/rename` so you can find sessions later with `--resume`

## Further Reading

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Keyboard Shortcuts Reference](https://docs.anthropic.com/en/docs/claude-code/shortcuts)
- [Issue Tracker](https://github.com/anthropics/claude-code/issues)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
