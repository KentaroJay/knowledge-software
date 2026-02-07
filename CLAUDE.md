# Development Learning Notes - Project Guide

## Project Overview

This is a personal knowledge base for software development concepts, patterns, and best practices. It's a collection of markdown notes organized by technology/domain.

## Repository Structure

```
├── CLAUDE.md              # This file - project-level Claude instructions
├── .claude/               # Claude Code configuration
│   ├── settings.local.json
│   ├── agents/            # Custom subagents
│   └── skills/            # Custom slash commands
├── claude-code/           # Documentation about Claude Code itself
├── neovim/                # Neovim configuration & Lua scripting notes
├── react-native/          # React Native development notes
├── lua/                   # Lua language notes
├── typescript/            # TypeScript notes
├── ios/                   # iOS development notes
└── general/               # General development concepts
```

## Note Format

All notes follow this structure (see @.claude/instructions.md for full details):

1. Title + one-line summary
2. Quick Summary (bullet points or table)
3. Context - when/why it matters
4. Explanation with code examples
5. Key Takeaways
6. Common Pitfalls
7. Further Reading
8. Metadata (date, project context)

## File Naming

- Use kebab-case: `npm-vs-cocoapods.md`
- Be descriptive: `state-management-patterns.md`

## Writing Style

- Clear and concise - no fluff
- Practical - real code examples
- Honest - document what worked AND what didn't
- Self-contained - no assumed context
- Use ASCII diagrams for architecture visuals

## Commands

```bash
# No build system - this is a markdown-only knowledge base
# Search notes by keyword
grep -r "keyword" .

# Browse by topic
ls -la neovim/
```

## Git Conventions

```bash
git commit -m "Add [topic]: [brief description]"
# Example: "Add React Native: npm vs CocoaPods comparison"
```

## When Creating Notes

- Check existing directories before creating new ones
- Follow the established note format in @.claude/instructions.md
- Reference `react-native/npm-vs-cocoapods.md` as the gold-standard template
- Add cross-references when topics relate: `See [Topic](../dir/file.md)`
- Include a README.md for new directories with a table of contents
