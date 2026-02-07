---
name: new-note
description: Create a new learning note following the repository's standard format
argument-hint: <directory/filename.md>
user-invocable: true
allowed-tools: Read, Write, Glob, Grep
---

Create a new learning note at the path: $ARGUMENTS

## Instructions

1. Read `.claude/instructions.md` for the full note format standards
2. Read `react-native/npm-vs-cocoapods.md` as a reference template
3. Check if the target directory exists; if not, create a README.md for it
4. Create the note following the standard structure:
   - Title + one-line summary
   - Quick Summary (bullets or table)
   - Context
   - Explanation with code examples
   - Real-World Example
   - Key Takeaways
   - Common Pitfalls
   - Further Reading
   - Metadata (date, project context)
5. Use proper markdown formatting with syntax-highlighted code blocks
6. Add cross-references to related existing notes

Ask the user what topic the note should cover if not obvious from the filename.
