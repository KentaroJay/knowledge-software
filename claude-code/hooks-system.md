# Hooks System

Automate deterministic actions at specific lifecycle events in Claude Code.

## Quick Summary

- Hooks are shell commands that run automatically at specific Claude Code events
- Unlike CLAUDE.md instructions (advisory), hooks **always execute** - they're guaranteed
- Can block actions (exit code 2), allow them (exit code 0), or just observe
- Configured in `settings.json` (project or user level)
- Three types: command, prompt-based, and agent-based

## Context

Sometimes you need guaranteed behavior - not just a suggestion Claude might follow. For example: always run Prettier after editing a file, block dangerous commands, or send a notification when Claude needs input. Hooks provide this deterministic layer on top of Claude's AI behavior.

## How It Works

### Hook Types

```
┌──────────────────────────────────────────────────┐
│  Command Hooks (type: "command")                 │
│  Run a shell command, communicate via exit codes │
│  Best for: formatting, linting, blocking         │
├──────────────────────────────────────────────────┤
│  Prompt Hooks (type: "prompt")                   │
│  Send data to Claude LLM for evaluation          │
│  Best for: nuanced policy decisions              │
├──────────────────────────────────────────────────┤
│  Agent Hooks (type: "agent")                     │
│  Spawn a subagent with tool access               │
│  Best for: complex validation requiring research │
└──────────────────────────────────────────────────┘
```

### Available Hook Events

| Event | When | Can Block? | Use Case |
|-------|------|-----------|----------|
| `SessionStart` | Session begins/resumes | No | Load env, inject context |
| `UserPromptSubmit` | Before processing prompt | Yes | Validate/filter prompts |
| `PreToolUse` | Before tool executes | Yes | Block dangerous operations |
| `PostToolUse` | After tool succeeds | No | Auto-format, lint |
| `PostToolUseFailure` | After tool fails | No | Log failures, alert |
| `Notification` | Claude needs input | No | Desktop notifications |
| `SubagentStart` | Subagent spawned | No | Setup for subagent |
| `SubagentStop` | Subagent finished | Yes | Cleanup, validation |
| `Stop` | Claude finishes responding | Yes | Verify completeness |
| `PreCompact` | Before context compaction | No | Re-inject critical context |
| `SessionEnd` | Session terminates | No | Cleanup, logging |

### Hook Communication

Hooks receive JSON via **stdin** and communicate back via **exit codes** and **stdout/stderr**:

```
┌─────────────┐    JSON stdin     ┌──────────────┐
│ Claude Code  │ ───────────────▶ │  Hook Script │
│              │                  │              │
│              │ ◀─── exit code ─ │  exit 0: OK  │
│              │ ◀─── stderr ──── │  exit 2: BLOCK│
│              │ ◀─── stdout ──── │  other: skip │
└─────────────┘                  └──────────────┘
```

**Input JSON example (PreToolUse):**
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf node_modules"
  }
}
```

## Real-World Examples

### 1. Desktop Notification When Claude Needs Input

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 2. Auto-Format Files After Editing

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 3. Block Destructive Commands

Create a script at `.claude/hooks/block-dangerous.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Block dangerous patterns
for pattern in "rm -rf /" "drop table" "force push" "> /dev/sda"; do
  if echo "$COMMAND" | grep -qi "$pattern"; then
    echo "Blocked: dangerous command pattern '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

Then reference it in settings:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-dangerous.sh"
          }
        ]
      }
    ]
  }
}
```

### 4. Inject Environment Variables at Session Start

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"$CLAUDE_ENV_FILE\" ]; then echo 'export NODE_ENV=development' >> \"$CLAUDE_ENV_FILE\"; fi"
          }
        ]
      }
    ]
  }
}
```

## Matchers

Matchers use regex to filter when hooks fire:

| Event | Matches On | Examples |
|-------|-----------|---------|
| `PreToolUse` / `PostToolUse` | Tool name | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | Start type | `startup`, `resume`, `compact` |
| `SessionEnd` | End reason | `clear`, `logout` |
| `Notification` | Notification type | `permission_prompt`, `idle_prompt` |
| `SubagentStart` / `SubagentStop` | Agent type | `Explore`, `Plan` |
| `UserPromptSubmit` / `Stop` | (no matcher) | Always fires |

## Configuration Locations

| Location | Scope | Git? |
|----------|-------|------|
| `~/.claude/settings.json` | All projects | No |
| `.claude/settings.json` | This project (team) | Yes |
| `.claude/settings.local.json` | This project (personal) | No |

## Key Takeaways

- Hooks are **deterministic** - they always run, unlike CLAUDE.md instructions
- Exit code 0 = allow, exit code 2 = block, anything else = continue
- Use `matcher` regex to target specific tools or events
- Keep hook scripts fast - they block Claude's execution
- Desktop notifications via `Notification` hook are very useful for long tasks
- Combine PreToolUse hooks for security with PostToolUse hooks for quality

## Common Pitfalls

- **Slow hooks** - A 10-second formatter hook will make every edit feel slow
- **Forgetting chmod +x** - Shell scripts need execute permission
- **Wrong exit codes** - Exit 1 means "error, continue anyway"; exit 2 means "block"
- **Missing jq** - Most hook scripts need `jq` to parse JSON input
- **Blocking too broadly** - A matcher of `""` matches everything; be specific

## Further Reading

- [Hooks Documentation](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Hook Examples Repository](https://github.com/anthropics/claude-code)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
