# Settings & Permissions

Configure Claude Code's access control, preferences, and behavior.

## Quick Summary

- Settings live in JSON files at user, project, and local levels
- Permissions use allow/deny/ask rules with tool-specific patterns
- Deny rules take priority, then ask, then allow
- Hooks, environment variables, and model preferences are also configured here
- Local settings (`.claude/settings.local.json`) are gitignored for personal preferences

## Context

By default, Claude Code asks permission for most actions. As you build trust, you can pre-approve safe operations (like `git status`) while explicitly blocking dangerous ones (like `rm -rf`). Settings files let you configure this and much more.

## Settings File Hierarchy

```
┌─────────────────────────────────────────────────┐
│  Managed (organization-wide, highest priority)  │
│  /Library/Application Support/ClaudeCode/       │
│  managed-settings.json                          │
│  (Cannot be overridden by users)                │
├─────────────────────────────────────────────────┤
│  Local (personal project overrides)             │
│  .claude/settings.local.json                    │
│  (Gitignored - your personal preferences)       │
├─────────────────────────────────────────────────┤
│  Project (team-shared)                          │
│  .claude/settings.json                          │
│  (Committed to git - team conventions)          │
├─────────────────────────────────────────────────┤
│  User (personal defaults)                       │
│  ~/.claude/settings.json                        │
│  (All your projects, lowest priority)           │
└─────────────────────────────────────────────────┘
```

**Precedence (highest to lowest):**
1. Managed settings (org-wide)
2. Command-line arguments
3. Local settings (`.claude/settings.local.json`)
4. Project settings (`.claude/settings.json`)
5. User settings (`~/.claude/settings.json`)

## Permission Rules

### Syntax

Rules follow the format: `Tool` or `Tool(specifier)`

```json
{
  "permissions": {
    "allow": [
      "Bash(git status*)",
      "Bash(npm run *)",
      "Read(.env*)"
    ],
    "deny": [
      "Bash(curl *)",
      "Bash(rm -rf *)"
    ],
    "ask": [
      "Bash(git push *)"
    ]
  }
}
```

### Rule Patterns

| Pattern | Matches |
|---------|---------|
| `Bash(npm test)` | Exact command |
| `Bash(npm run *)` | Any npm run command |
| `Bash(git *)` | Any git command |
| `Read(.env*)` | Any .env file |
| `WebFetch(domain:example.com)` | Specific domain |
| `Edit` | Any edit operation |

### Evaluation Order

```
Request: Bash("npm test")
         │
         ▼
   ┌─── Deny rules ──┐
   │ Match? ──▶ BLOCK │
   └──────────────────┘
         │ No match
         ▼
   ┌─── Ask rules ───┐
   │ Match? ──▶ PROMPT│
   └──────────────────┘
         │ No match
         ▼
   ┌─── Allow rules ──┐
   │ Match? ──▶ ALLOW  │
   └──────────────────┘
         │ No match
         ▼
      DEFAULT: PROMPT
```

First matching rule in each category wins.

## Real-World Examples

### 1. Safe Defaults for a Markdown Repository

```json
{
  "permissions": {
    "allow": [
      "Bash(tree:*)",
      "Bash(git status*)",
      "Bash(git log*)",
      "Bash(git diff*)",
      "Bash(git branch*)",
      "Bash(git checkout*)",
      "Bash(ls *)"
    ]
  }
}
```

### 2. Web Project with Build Tools

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(npm test*)",
      "Bash(npx prettier *)",
      "Bash(npx eslint *)",
      "Bash(git *)"
    ],
    "deny": [
      "Bash(npm publish*)",
      "Bash(rm -rf /)*",
      "Read(.env*)"
    ],
    "ask": [
      "Bash(git push *)",
      "Bash(npm install *)"
    ]
  }
}
```

### 3. Full Settings File

```json
{
  "permissions": {
    "allow": [
      "Bash(git status*)",
      "Bash(npm test*)"
    ],
    "deny": [
      "Read(.env*)"
    ]
  },
  "env": {
    "NODE_ENV": "development"
  },
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs input\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

## Available Settings Keys

| Key | Type | Description |
|-----|------|-------------|
| `permissions` | object | Allow/deny/ask rules |
| `env` | object | Environment variables |
| `model` | string | Default AI model |
| `hooks` | object | Lifecycle hooks config |
| `sandbox` | object | Sandboxing settings |
| `outputStyle` | string | Response formatting |
| `language` | string | Response language |

## Managing Permissions Interactively

Inside Claude Code:
```
/permissions                    # View and edit permission rules
/config                         # Open settings interface
Shift+Tab                       # Cycle permission modes
```

### Permission Modes (Shift+Tab)

| Mode | Behavior |
|------|----------|
| **Default** | Ask for most actions |
| **Accept Edits** | Auto-approve file edits, ask for others |
| **Don't Ask** | Auto-approve most actions |
| **Plan** | Read-only, no modifications |

## Key Takeaways

- Use `.claude/settings.local.json` for personal preferences (gitignored)
- Use `.claude/settings.json` for team-shared conventions (committed)
- Deny rules always win over allow rules
- Start restrictive and add allow rules as you build trust
- Use wildcard patterns (`*`) for flexible matching
- `Shift+Tab` cycles permission modes quickly during a session

## Common Pitfalls

- **Too permissive** - `"allow": ["Bash(*)]"` approves everything including `rm -rf`
- **Conflicting rules** - Deny always wins; an allow rule can't override a deny
- **Forgetting local vs project** - `settings.local.json` won't affect your teammates
- **Stale rules** - Remove allow rules for tools you no longer use
- **Not using ask** - For semi-dangerous commands, `ask` is better than `allow` or `deny`

## Further Reading

- [Settings Documentation](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Permissions Guide](https://docs.anthropic.com/en/docs/claude-code/permissions)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
