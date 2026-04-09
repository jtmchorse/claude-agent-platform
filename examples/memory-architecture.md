# Memory Architecture

Three-layer shared state system enabling multi-agent coordination.

## Layer 1: Knowledge Graph (Persistent, Structured)

Entities and relations stored via MCP memory server. Used for:
- Service registry (IPs, ports, API patterns, gotchas)
- Architecture decisions (what was chosen and why)
- Stable patterns (validated approaches)
- People context (roles, preferences)

**Written by:** `/end-session --full`, `/retro`
**Read by:** All skills at session start, research for prior knowledge

## Layer 2: Vault (Persistent, Documents)

Markdown files organized by PARA method (Projects, Areas, Resources, Archives):
- Research reports with trust-scored sources
- Daily notes with focus session logs
- Project tracking and progress documentation
- Shared reports for cross-session work

**Written by:** `/research`, `/briefing`, `/focus`, subagents
**Read by:** All skills, morning briefing, research prior-art checks

## Layer 3: Session Memory (Conversation-Scoped)

MEMORY.md loaded at conversation start. Contains:
- Key paths and conventions
- User preferences
- Model selection rules
- Quick lookups (pointers to knowledge graph)

**Written by:** `/end-session`
**Read by:** Automatically injected into every session

## How They Interact

```
Session starts
  → Load MEMORY.md (Layer 3)
  → Load recent session learnings
  → Knowledge graph available via MCP (Layer 1)
  → Vault available via direct file access (Layer 2)

During session
  → Skills read from all three layers
  → Skills write to Layer 2 (vault) directly
  → Layer 1 updates deferred to end-session

Session ends (/end-session)
  → Capture learnings → Layer 2 (session-learnings.md)
  → Update knowledge graph → Layer 1 (--full mode)
  → Update MEMORY.md → Layer 3
  → Sync and commit
```

## Why Three Layers?

- **Knowledge graph** is fast to query but expensive to update (requires careful entity management). Good for facts that change slowly.
- **Vault** is easy to write but slow to search exhaustively. Good for detailed documents and reports.
- **Session memory** is tiny but always present. Good for routing decisions and preferences.

A research agent doesn't need your full project history — it needs to know where to save its report. Session memory handles that. A retro skill needs all your decisions from the past month — the knowledge graph handles that. A briefing skill needs today's meal plan — the vault handles that.
