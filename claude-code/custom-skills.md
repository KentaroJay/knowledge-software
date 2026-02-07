# Custom Skills (Slash Commands)

Create reusable workflows invoked with `/command-name` in Claude Code.

## Quick Summary

- Skills are custom slash commands defined in `.claude/skills/<name>/SKILL.md`
- Invoked with `/skill-name` or automatically when Claude detects a matching task
- Support arguments via `$ARGUMENTS` placeholder
- Can restrict tool access, override model, and include hooks
- Project-scoped (`.claude/skills/`) or user-scoped (`~/.claude/skills/`)

## Context

You often repeat the same type of request: "create a new note following our format", "review this code", "search for X". Skills let you package these as reusable one-line commands instead of typing the full prompt every time.

## How It Works

### Skill File Structure

```
.claude/skills/
├── new-note/
│   └── SKILL.md           # Required - instructions + config
├── review-note/
│   └── SKILL.md
└── search-notes/
    └── SKILL.md
```

Each skill needs at minimum a `SKILL.md` file with YAML frontmatter and markdown instructions.

### Anatomy of a SKILL.md

```markdown
---
name: my-skill                        # Unique identifier
description: What this skill does     # When to auto-invoke
argument-hint: <arg-description>      # Autocomplete hint
user-invocable: true                  # Show in /menu
allowed-tools: Read, Write, Glob      # Restrict tools
model: sonnet                         # Override model
---

Instructions for Claude when this skill is invoked.

Use $ARGUMENTS to reference what the user passed.
Use $ARGUMENTS[0] or $0 for the first argument.
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique ID, used as `/name` |
| `description` | Yes | When to use this skill |
| `argument-hint` | No | Hint shown in autocomplete |
| `user-invocable` | No | If `false`, only auto-invoked |
| `disable-model-invocation` | No | If `true`, only manual `/name` |
| `allowed-tools` | No | Comma-separated tool whitelist |
| `model` | No | `sonnet`, `opus`, `haiku` |
| `context` | No | `fork` runs in subagent |
| `agent` | No | Subagent type (`Explore`, `Plan`) |
| `hooks` | No | Skill-scoped lifecycle hooks |

## Real-World Examples

### 1. Create a New Learning Note

```markdown
# .claude/skills/new-note/SKILL.md

---
name: new-note
description: Create a new learning note following repository standards
argument-hint: <directory/filename.md>
user-invocable: true
allowed-tools: Read, Write, Glob, Grep
---

Create a new learning note at: $ARGUMENTS

1. Read `.claude/instructions.md` for format standards
2. Read `react-native/npm-vs-cocoapods.md` as reference
3. Create the note with all required sections
4. Add cross-references to related notes
```

**Usage:**
```
/new-note typescript/generics-patterns.md
```

### 2. Review a Note

```markdown
# .claude/skills/review-note/SKILL.md

---
name: review-note
description: Review a note for quality and completeness
argument-hint: <path/to/note.md>
user-invocable: true
allowed-tools: Read, Glob, Grep
---

Review the note at $ARGUMENTS against our quality checklist.
Output: strengths, missing sections, and suggestions.
```

**Usage:**
```
/review-note neovim/plugin-management.md
```

### 3. Fix a GitHub Issue

```markdown
# .claude/skills/fix-issue/SKILL.md

---
name: fix-issue
description: Investigate and fix a GitHub issue
argument-hint: <issue-number>
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

Fix GitHub issue #$ARGUMENTS:

1. Run `gh issue view $0` to get details
2. Investigate the codebase for the root cause
3. Implement the fix
4. Write or update tests
5. Create a commit with the fix
```

**Usage:**
```
/fix-issue 42
```

### 4. Quick Research (Runs in Subagent)

```markdown
# .claude/skills/research/SKILL.md

---
name: research
description: Research a topic without modifying files
argument-hint: <topic>
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob, WebSearch, WebFetch
---

Research $ARGUMENTS thoroughly and report findings.
```

**Usage:**
```
/research "React Server Components best practices"
```

## Skill Locations

| Location | Scope | Shared? |
|----------|-------|---------|
| `.claude/skills/<name>/SKILL.md` | Current project | Yes (git) |
| `~/.claude/skills/<name>/SKILL.md` | All projects | No |

## String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed to the skill |
| `$ARGUMENTS[0]` or `$0` | First argument |
| `$ARGUMENTS[1]` or `$1` | Second argument |
| `${CLAUDE_SESSION_ID}` | Current session ID |

## Key Takeaways

- Skills turn repetitive prompts into one-line slash commands
- Keep skill instructions focused and specific
- Use `allowed-tools` to restrict what a skill can do (principle of least privilege)
- Use `context: fork` for read-only research tasks (runs in isolated subagent)
- Project skills go in `.claude/skills/` and are shared via git
- User skills go in `~/.claude/skills/` for personal cross-project use

## Common Pitfalls

- **Missing frontmatter** - SKILL.md must have `---` delimited YAML at the top
- **Name conflicts** - Project and user skills with same name may collide
- **Overly broad tools** - Don't give `Bash` access to a read-only skill
- **Vague instructions** - Be specific about what Claude should do step by step
- **No argument hint** - Users won't know what to pass without `argument-hint`

## Further Reading

- [Claude Code Skills Documentation](https://docs.anthropic.com/en/docs/claude-code/skills)
- [Slash Commands Reference](https://docs.anthropic.com/en/docs/claude-code)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
