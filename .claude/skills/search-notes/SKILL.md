---
name: search-notes
description: Search across all learning notes for a topic, keyword, or concept
argument-hint: <search-term>
user-invocable: true
allowed-tools: Read, Glob, Grep
---

Search all learning notes for: $ARGUMENTS

## Instructions

1. Search file names for matches using Glob
2. Search file contents for matches using Grep
3. Read relevant sections from matching files
4. Present results grouped by directory/topic:
   - File path and title
   - Matching excerpt (2-3 lines of context)
   - Relevance to the search term
5. Suggest related topics the user might also want to explore
