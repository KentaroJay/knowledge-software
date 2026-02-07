# Custom Agents (Subagents)

Create specialized AI assistants that Claude delegates tasks to autonomously.

## Quick Summary

- Agents are specialized Claude instances with custom instructions, tool access, and models
- Defined in `.claude/agents/<name>.md` with YAML frontmatter
- Claude automatically delegates tasks matching the agent's description
- Can also be invoked manually or via skills
- Support persistent memory across sessions

## Context

Some tasks need a specialist: a security reviewer who only reads code, a research assistant who searches the web, or a code reviewer who follows specific standards. Agents let you define these specialists once and Claude will automatically delegate to them when appropriate.

## How It Works

### Built-in Agents

Claude Code comes with several built-in subagent types:

```
┌──────────────────────────────────────────────┐
│  Explore (haiku)                             │
│  Fast, read-only codebase exploration        │
│  Tools: Read, Grep, Glob (no Write/Edit)     │
├──────────────────────────────────────────────┤
│  Plan (inherits model)                       │
│  Safe analysis for planning mode             │
│  Tools: Read-only                            │
├──────────────────────────────────────────────┤
│  General-purpose (inherits model)            │
│  Full-featured for complex tasks             │
│  Tools: All tools available                  │
└──────────────────────────────────────────────┘
```

### Custom Agent File Structure

```
.claude/agents/
├── note-reviewer.md       # Reviews notes for quality
└── research-assistant.md  # Researches topics for new notes
```

### Anatomy of an Agent File

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability.
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit
model: sonnet
---

You are a senior code reviewer. When invoked:

1. Read the files to review
2. Check for security issues
3. Evaluate code clarity
4. Provide actionable feedback

## Review Criteria
- Security vulnerabilities
- Performance concerns
- Code readability
- Test coverage
```

### Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique identifier |
| `description` | string | When Claude should delegate to this agent |
| `tools` | string | Comma-separated allowed tools |
| `disallowedTools` | string | Comma-separated denied tools |
| `model` | string | `opus`, `sonnet`, `haiku`, or `inherit` |
| `permissionMode` | string | `default`, `acceptEdits`, `plan` |
| `skills` | array | Skills to preload |
| `memory` | string | `user`, `project`, or `local` |
| `hooks` | object | Agent-scoped lifecycle hooks |

## Real-World Examples

### 1. Note Reviewer (Read-Only)

```markdown
# .claude/agents/note-reviewer.md

---
name: note-reviewer
description: Reviews learning notes for quality and completeness
tools: Read, Grep, Glob
disallowedTools: Write, Edit, Bash
model: haiku
---

You are a note reviewer. Check notes against:
- [ ] Clear title and summary
- [ ] Quick Summary section
- [ ] Code examples with syntax highlighting
- [ ] Key Takeaways
- [ ] Common Pitfalls
- [ ] Proper markdown formatting
```

**How it's used:**
- Claude automatically delegates when you say "review this note"
- Uses Haiku (fast, cheap) since it's just reading and evaluating
- Cannot modify files - only reports findings

### 2. Research Assistant (Web Access)

```markdown
# .claude/agents/research-assistant.md

---
name: research-assistant
description: Researches topics to help create learning notes
tools: Read, Grep, Glob, WebSearch, WebFetch
disallowedTools: Write, Edit
model: sonnet
---

Research the given topic and structure findings to match
the repository's note format. Include:
- Official documentation links
- Practical code examples
- Common pitfalls
- Version-specific notes
```

### 3. Security Auditor with Persistent Memory

```markdown
# .claude/agents/security-auditor.md

---
name: security-auditor
description: Audits code for security vulnerabilities
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit
model: opus
memory: project
---

You are a security auditor. Check for OWASP Top 10:
- Injection flaws
- Broken authentication
- Sensitive data exposure
- XSS vulnerabilities
- Insecure dependencies

Save findings to your memory for tracking across sessions.
```

**Memory persistence:**
- `memory: user` → `~/.claude/agent-memory/security-auditor/`
- `memory: project` → `.claude/agent-memory/security-auditor/`
- `memory: local` → `.claude/agent-memory-local/security-auditor/`

## How Delegation Works

```
┌─────────────┐   "Review this note"   ┌─────────────┐
│    User      │ ────────────────────▶  │   Claude     │
└─────────────┘                        │   (Main)     │
                                       └──────┬──────┘
                                              │
                            Matches description│
                            of note-reviewer   │
                                              ▼
                                       ┌─────────────┐
                                       │ note-reviewer│
                                       │  (Subagent)  │
                                       │  model:haiku │
                                       │  read-only   │
                                       └──────┬──────┘
                                              │
                                              ▼
                                       Returns review
                                       to main Claude
```

1. User makes a request
2. Claude checks if any agent's `description` matches
3. If yes, spawns the agent as a subprocess
4. Agent works autonomously with its restricted tools
5. Returns results to the main Claude session

## Agent Locations

| Location | Scope | Shared? |
|----------|-------|---------|
| `.claude/agents/<name>.md` | Current project | Yes (git) |
| `~/.claude/agents/<name>.md` | All projects | No |

## Key Takeaways

- Agents are specialists - give them focused descriptions and restricted tools
- Use `disallowedTools` to enforce read-only or write-only behavior
- Choose the right model: Haiku for fast reads, Sonnet for complex analysis, Opus for critical decisions
- Persistent memory lets agents learn and track findings across sessions
- Claude auto-delegates based on the `description` field - make it specific

## Common Pitfalls

- **Too broad description** - "Helps with everything" means Claude delegates too often
- **Too many tools** - Give agents only what they need (principle of least privilege)
- **Wrong model** - Using Opus for a simple grep task wastes time and money
- **No description** - Without a clear description, Claude won't know when to delegate
- **Memory bloat** - Persistent memory grows over time; review and clean periodically

## Further Reading

- [Claude Code Agents Documentation](https://docs.anthropic.com/en/docs/claude-code/agents)
- [Agent SDK](https://docs.anthropic.com/en/docs/agents/agent-sdk)

---

*Created: February 2026*
*Project Context: dev-learning-notes repository setup*
