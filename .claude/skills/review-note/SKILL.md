---
name: review-note
description: Review a learning note for quality and completeness against repository standards
argument-hint: <path/to/note.md>
user-invocable: true
allowed-tools: Read, Glob, Grep
---

Review the learning note at: $ARGUMENTS

## Instructions

1. Read the note format standards from `.claude/instructions.md`
2. Read `react-native/npm-vs-cocoapods.md` as the gold-standard reference
3. Read the note to review
4. Evaluate against the quality checklist:
   - [ ] Clear title and one-line summary
   - [ ] Quick Summary section
   - [ ] Context section
   - [ ] Detailed explanation with code examples
   - [ ] Real-world examples
   - [ ] Key Takeaways
   - [ ] Common Pitfalls
   - [ ] Further Reading
   - [ ] Metadata (date, project context)
   - [ ] Proper markdown formatting
   - [ ] Code blocks with syntax highlighting
   - [ ] ASCII diagrams where helpful
   - [ ] Cross-references to related topics
   - [ ] Kebab-case filename
5. Output a structured review with strengths, missing sections, and actionable suggestions
