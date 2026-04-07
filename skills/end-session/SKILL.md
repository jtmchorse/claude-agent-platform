---
name: end-session
description: Session close protocol — captures learnings, syncs state, ensures nothing is left uncommitted. Entry point for the self-improvement loop.
---

# End Session

Close out the current session by capturing what was learned and ensuring all work is saved.

**Modes:**
- `/end-session` — **fast mode** (default). Write learnings, sync, commit. ~2 min.
- `/end-session --full` — all steps including knowledge graph, learning extractor, pattern promotion. ~8 min.

---

# Fast Mode (default)

## Step 1: Gather Session Context

```bash
git diff --stat HEAD
git status -s
git log --oneline -10 --since="8 hours ago"
```

## Step 2: Capture Learnings

Append 2-4 bullet points to `session-learnings.md`. Each bullet should be:

- **What worked** — a technique or approach that saved time
- **What broke** — a gotcha or friction point encountered
- **What to try next** — an idea sparked but not pursued

Format:
```
## YYYY-MM-DD — [Brief topic/feature name]

- [domain] [learning 1]
- [domain] [learning 2]
```

**Rules:**
- Be specific, not generic. "MCP SSH swallows stderr — always append `2>&1`" beats "learned about SSH"
- Include file paths or commands when relevant
- If a pattern appeared for the 2nd+ time, flag it: `[PROMOTE]`

## Step 3: Sync & Save

```bash
# Sync task tracker
bd sync 2>/dev/null || true

# Check git state
git status
```

Ask the user before committing — show what would be committed and where it would push.

## Step 4: Report

```
Learnings: [count] captured
Git: [clean / N files uncommitted]
```

---

# Full Mode (`--full`)

All fast mode steps, PLUS:

## Full Step A: Decision Detection

Scan learnings for "chose", "decided", "because", "instead of". If found, record in:
- Knowledge graph as a decision entity
- `decisions.md` with context, rationale, and alternatives considered

## Full Step B: Update Knowledge Graph

For each significant learning:
1. Search for existing entities
2. Add observations to existing entities
3. Create entities for new tools/services/patterns
4. Wire relations between entities

## Full Step C: Learning Extractor

Classify each learning into categories (see [learning-extractor](../learning-extractor/) skill):
- Config fixes → settings patches
- Operational principles → CLAUDE.md updates
- Tool preferences → documentation
- Service gotchas → knowledge graph observations
- Skill candidates → draft new skills

## Full Step D: Promote Stable Patterns

Scan for `[PROMOTE]` tags. Also scan last 5 sessions for repeated themes.
If a pattern appeared 2+ times, add to `patterns.md`.

## Full Step E: Settings Hygiene

Scan config for stale one-off entries. Report count, only clean if user approves.

## Full Mode Report

```
Learnings: [count] captured
Graph: [N] entities updated/created
Patterns promoted: [count]
Decisions: [count]
Settings: [N entries, M stale]
Git: [clean / N files uncommitted]
```
