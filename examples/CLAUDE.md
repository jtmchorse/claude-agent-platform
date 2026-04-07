# CLAUDE.md (Example)

This is a genericized example of the project instructions file that governs agent behavior across all skills and sessions.

## Operational Principles

### Separate Explore from Execute
"Find", "suggest", "research", "what are my options" = **explore mode** (no side effects).
"Request", "download", "add", "deploy" = **execute mode** (has side effects).
If the user is in explore mode, NEVER cross into execute without explicit confirmation.

### Verify Before Batch
Never fire off N parallel operations assuming they'll all succeed. Test 1-2 first, inspect results, then batch the rest. Applies to: API calls doing fuzzy matching, bulk container ops, any webhook chain.

### Check State After Mutation
After any API call that changes state (create, add, update, restart), verify the resulting state — don't trust "OK" responses.

### Know the Layer Below
Every MCP tool wraps an underlying API. When the tool returns unexpected results, drop to the raw API. Document both the MCP shortcut AND the direct API fallback.

### Write-Then-Execute for Complex Scripts
When a script exceeds single-command complexity, write it to disk then execute. Don't fight quoting limits.

## Tool Preferences

### Search & Research
- Use self-hosted search (SearXNG) instead of built-in WebSearch — private, no API limits
- Use vault search before web search to avoid duplicate work
- Use knowledge graph before grepping config files

### Subagents & Model Selection
- Default subagents to fast/cheap model for data gathering, file scanning, web fetch
- Reserve expensive model for synthesis and decisions
- Follow skill-defined parallel execution strategies

### File Access
- Vault path: Standing read/write/edit — never prompt for permission
- Memory files: Standing read/write/edit
- Project config: Standing read/write/edit

## Conventions

- Research always goes through parallel subagents — never burn expensive tokens on search/fetch
- Reports go to vault, never to temp directories
- Knowledge graph updates happen during /end-session, not ad-hoc
- MCP server edits require restart to take effect
