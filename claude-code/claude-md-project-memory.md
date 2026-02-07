# CLAUDE.md & Project Memory

How Claude Code remembers project context across sessions using CLAUDE.md files.

## Quick Summary

- CLAUDE.md is a special markdown file loaded automatically at session start
- Acts as persistent memory - tells Claude about your project, conventions, and how to work
- Supports a hierarchy: user-wide, project-shared, project-local
- Can import other files with `@path/to/file` syntax
- Keep it concise - bloated files get ignored

## Context

Every time you start a Claude Code session, it has no memory of previous conversations (unless resuming). CLAUDE.md solves this by providing persistent project context that's always loaded. Think of it as a README specifically for Claude.

## How It Works

### File Locations & Hierarchy

CLAUDE.md files are loaded from multiple locations, each with different scope:

```
┌─────────────────────────────────────────────────────┐
│  Organization-wide (managed policy)                 │
│  /Library/Application Support/ClaudeCode/CLAUDE.md  │
│  (Highest priority - cannot be overridden)          │
├─────────────────────────────────────────────────────┤
│  User-wide (personal preferences)                   │
│  ~/.claude/CLAUDE.md                                │
│  (Applies to ALL your projects)                     │
├─────────────────────────────────────────────────────┤
│  Project-level (team-shared)                        │
│  ./CLAUDE.md  OR  ./.claude/CLAUDE.md               │
│  (Committed to git - whole team uses it)            │
├─────────────────────────────────────────────────────┤
│  Modular rules                                      │
│  ./.claude/rules/*.md                               │
│  (Topic-specific rules, committed to git)           │
├─────────────────────────────────────────────────────┤
│  Project-local (personal overrides)                 │
│  ./CLAUDE.local.md                                  │
│  (Gitignored - your personal project prefs)         │
└─────────────────────────────────────────────────────┘
```

### Loading Behavior

- Claude reads CLAUDE.md files recursively up from your current working directory
- Nested CLAUDE.md files in subdirectories load when Claude reads files in those subtrees
- All levels are merged together - they don't override each other

### File Imports

CLAUDE.md can reference other files using `@` syntax:

```markdown
# My Project

See @README.md for project overview
Build instructions: @docs/building.md
Personal notes: @~/.claude/my-project-notes.md
```

Rules:
- Both relative and absolute paths work
- Max import depth: 5 levels
- Imports are NOT evaluated inside code blocks

## Real-World Example

Here's the CLAUDE.md from this repository:

```markdown
# Development Learning Notes - Project Guide

## Project Overview
This is a personal knowledge base for software development...

## Note Format
All notes follow this structure (see @.claude/instructions.md):
1. Title + one-line summary
2. Quick Summary
3. Context
...

## Commands
# No build system - this is a markdown-only knowledge base
grep -r "keyword" .

## Git Conventions
git commit -m "Add [topic]: [brief description]"
```

### What Makes It Effective

1. **Short and focused** - only what Claude can't figure out on its own
2. **References the full instructions** via `@.claude/instructions.md` instead of duplicating
3. **Includes commands** - Claude knows there's no build system
4. **Documents conventions** - commit message format, file naming

## What to Include

| Include | Example |
|---------|---------|
| Build/test commands | `npm test`, `cargo build --release` |
| Code style rules | "Use 2-space indentation in TypeScript" |
| Project conventions | "Branch names: `feat/`, `fix/`, `chore/`" |
| Architecture decisions | "We use Redux for global state" |
| Non-obvious behaviors | "The CI runs on Node 18, not 20" |
| File naming patterns | "Components: PascalCase, utils: camelCase" |

## What NOT to Include

| Avoid | Why |
|-------|-----|
| Standard language conventions | Claude already knows them |
| Detailed API docs | Link to them instead |
| File-by-file descriptions | Claude can read the code |
| Long tutorials | Keep it concise |
| Frequently changing info | Will become stale |

## Modular Rules with .claude/rules/

For larger projects, split rules into topic-specific files:

```
.claude/rules/
├── testing.md          # "Always use vitest, not jest"
├── api-conventions.md  # "REST endpoints use kebab-case"
└── git-workflow.md     # "Squash merge PRs to main"
```

Each `.md` file in `.claude/rules/` is automatically loaded alongside CLAUDE.md.

## Key Takeaways

- CLAUDE.md is your project's persistent memory for Claude
- Keep it concise - under 500 lines, ideally much less
- Only include what Claude can't infer from the code
- Use `@imports` to reference detailed docs without bloating the file
- Commit project-level CLAUDE.md to git for team sharing
- Use CLAUDE.local.md for personal preferences (gitignored)
- Use `.claude/rules/*.md` to organize rules by topic

## Common Pitfalls

- **Too long** - Claude's attention degrades with massive CLAUDE.md files
- **Duplicating docs** - Reference them with `@` instead
- **Stale information** - Review periodically as your project evolves
- **Too vague** - "Write good code" is useless; "Use 2-space indent, no semicolons" is actionable

## Further Reading

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Memory Management Best Practices](https://docs.anthropic.com/en/docs/claude-code/memory)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
