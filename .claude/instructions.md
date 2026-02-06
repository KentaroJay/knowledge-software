# Development Learning Notes - Claude Instructions

## Repository Purpose

This is a **personal knowledge base** for software development concepts, patterns, and solutions learned through real projects.

## Goals

1. **Document learnings** from hands-on development
2. **Create reusable references** for future projects
3. **Build a searchable knowledge base** of solutions
4. **Track growth** as a developer
5. **Share knowledge** with future self and others

## Organization Principles

### Directory Structure
- **Organize by technology or domain** (e.g., `react-native/`, `typescript/`, `neovim/`)
- **Use clear, descriptive directory names** that reflect the content
- **One directory per major topic area** - Keep related content together
- **Subdirectories for related sub-topics** when a topic grows complex
- **Keep structure flat when possible** - Avoid excessive nesting

### How to Organize
- Browse existing directories to see established patterns
- Create new directories for major technologies or domains
- Group related concepts under the same parent directory
- Use README.md files to explain directory contents when needed

## Note Format Standards

### File Naming
- Use kebab-case: `npm-vs-cocoapods.md`
- Be descriptive: `state-management-patterns.md`
- Include context: `react-navigation-tab-setup.md`

### Note Structure

Each note should include:

1. **Title and Summary** - Quick overview at top
2. **Context** - When/why this is useful
3. **Explanation** - Clear concept explanation
4. **Examples** - Real code examples (preferably from actual projects)
5. **Diagrams** - Visual aids where helpful (ASCII art or descriptions)
6. **Key Takeaways** - Bullet points of main concepts
7. **Common Pitfalls** - Things to watch out for
8. **Further Reading** - Links to resources
9. **Metadata** - When created, project context

### Markdown Format

```markdown
# Topic Title

Brief one-sentence summary.

## Quick Summary
- Key point 1
- Key point 2
- Key point 3

## Context
When and why this matters...

## Explanation
Detailed explanation with examples...

## Real-World Example
From Project X...

## Key Takeaways
- Takeaway 1
- Takeaway 2

## Common Pitfalls
- Pitfall 1
- Pitfall 2

## Further Reading
- [Link](url)

---
*Created: [Date]*
*Project Context: [Project name]*
```

## Writing Guidelines

### Tone and Style
- **Clear and concise** - Get to the point
- **Practical** - Include real examples
- **Honest** - Document what worked AND what didn't
- **Future-proof** - Write for yourself in 6 months
- **Complete** - Self-contained, no assumed context

### What to Include
- ✅ Real examples from projects
- ✅ Why something matters
- ✅ Common mistakes
- ✅ Visual diagrams
- ✅ Commands and code snippets
- ✅ Context and rationale

### What to Avoid
- ❌ Overly verbose explanations
- ❌ Copying documentation verbatim
- ❌ Outdated version-specific info (note versions if included)
- ❌ Assuming prior knowledge without explanation
- ❌ Incomplete examples

## Adding New Notes

### Process
1. **Identify the concept** - What did you learn?
2. **Choose category** - Which directory?
3. **Create file** - Use kebab-case naming
4. **Write content** - Follow format above
5. **Add examples** - From actual project code
6. **Review** - Is it clear? Complete?
7. **Commit** - Git commit with descriptive message

### Commit Message Format
```bash
git commit -m "Add [topic]: [brief description]"

# Examples:
git commit -m "Add React Native: npm vs CocoaPods comparison"
git commit -m "Add TypeScript: Generic constraints pattern"
git commit -m "Add iOS: CocoaPods installation on Apple Silicon"
```

## Cross-References

When topics relate to each other, add cross-references:

```markdown
## Related Topics
- See [State Management Patterns](../react-native/state-management.md)
- Related to [TypeScript Generics](../typescript/generics.md)
```

## Code Examples

### Format
- Use syntax highlighting: \`\`\`typescript, \`\`\`bash, etc.
- Include comments for clarity
- Keep examples focused and minimal
- Show both good and bad examples when helpful

### Example Format
```markdown
### Good Example
\`\`\`typescript
// Clear, type-safe implementation
interface User {
  id: string;
  name: string;
}
\`\`\`

### Bad Example (Don't Do This)
\`\`\`typescript
// Using 'any' loses type safety
const user: any = {...};
\`\`\`
```

## Diagrams and Visuals

Use ASCII art or text-based diagrams for simple visuals:

```
┌─────────────────┐
│   Component A   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Component B   │
└─────────────────┘
```

For complex diagrams, describe in words or link to images.

## Maintenance

### Updating Notes
- Add "Updated: [Date]" at bottom when making changes
- Keep original creation date
- Document what changed and why

### Reviewing Notes
- Periodically review for outdated info
- Update examples if better approaches emerge
- Archive outdated patterns (don't delete, mark as deprecated)

## Search and Discovery

### Finding Information
```bash
# Search by keyword
grep -r "keyword" .

# Find all notes about a topic
find . -name "*topic*"

# Search within category
grep -r "pattern" react-native/
```

### Tags (in frontmatter if desired)
```markdown
---
tags: [react-native, performance, optimization]
difficulty: intermediate
---
```

## Quality Checklist

Before committing a new note, verify:

- [ ] Clear title and summary
- [ ] Real-world example included
- [ ] Key takeaways listed
- [ ] Common pitfalls documented
- [ ] Code examples have syntax highlighting
- [ ] No spelling/grammar errors
- [ ] Links work (if any)
- [ ] Proper markdown formatting
- [ ] Metadata included (date, project)

## Project Context

Many notes will reference specific projects. Include context:

```markdown
*Project Context: Built during HabitTracker app (React Native habit tracking app)*
```

This helps future-you remember where the knowledge came from.

## Privacy and Sharing

- This is a **private learning repository**
- Can be made public later if desired
- Don't include sensitive information (API keys, passwords)
- Don't include proprietary company code
- Personal projects and open-source examples are fine

## Claude's Role

When creating new notes in this repository:

1. **Follow the format** - Use the structure defined above
2. **Be comprehensive** - Include all sections
3. **Use real examples** - From actual project context when available
4. **Add visuals** - ASCII diagrams for architecture
5. **Cross-reference** - Link to related topics
6. **Stay practical** - Focus on actionable knowledge

## Example Note Reference

See `react-native/npm-vs-cocoapods.md` as a template for:
- Structure and formatting
- Level of detail
- Example inclusion
- Visual diagrams
- Practical focus

## Growth Tracking

As the repository grows, consider:
- Adding a CHANGELOG.md
- Creating an index of topics
- Organizing by difficulty level
- Adding timestamps to track learning pace

---

**Remember:** The best notes are the ones you'll actually use later. Write for future-you!

*Created: February 2026*
