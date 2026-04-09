# Pattern: Context Engineering

Managing what an AI agent knows, when it knows it, and how that knowledge persists across sessions.

## The Problem

The context window is finite. With 110+ tools loaded, you've already spent significant tokens before the first message. Add conversation history, file contents, and tool results — the window fills fast. What the agent knows at any given moment determines the quality of its output more than which model you're using.

Context engineering is the discipline of making sure the right information is available at the right time without wasting capacity on information that isn't relevant.

## The Three-Layer Architecture

Different types of knowledge have different access patterns:

```
┌─────────────────────────────────────────────┐
│  Layer 3: Session Memory                     │
│  Always loaded. Tiny. Routing decisions,     │
│  user preferences, active conventions.       │
│  ~2-4K tokens.                               │
├─────────────────────────────────────────────┤
│  Layer 2: Document Store (Vault)             │
│  Loaded on demand. Reports, research,        │
│  specs, daily notes. Searchable.             │
│  ~500+ files, accessed via search tools.     │
├─────────────────────────────────────────────┤
│  Layer 1: Knowledge Graph                    │
│  Structured entities and relations.          │
│  Services, patterns, decisions, people.      │
│  Queried by keyword, returns focused data.   │
└─────────────────────────────────────────────┘
```

### Layer 3: Session Memory (always present)

A small markdown file (~2-4K tokens) loaded at the start of every session. Contains:
- User preferences and workflow conventions
- Active project context
- Pointers to where to find deeper information
- Recent session learnings (last 3-4 sessions only)

**Design principle:** If it's not needed in 80%+ of sessions, it doesn't belong here. Everything else is "deferred context" loaded on demand.

### Layer 2: Document Store (on demand)

Markdown files organized for human readability and machine searchability. Accessed via search tools, not loaded wholesale.

The agent checks the document store when it needs:
- Prior research on a topic (before doing new web searches)
- Project-specific context (architecture docs, specs)
- Historical data (meeting notes, decision records)

**Design principle:** The agent should check what it already knows before going to external sources. A "check research index first" rule prevents redundant work.

### Layer 1: Knowledge Graph (structured queries)

Entities and relations for facts that change slowly: service configurations, architectural decisions, known gotchas, team members. Queried by keyword, returns focused results.

**Design principle:** If it's a fact that multiple skills need, it goes in the graph. If it's a narrative or analysis, it goes in the document store. Don't put essays in the graph. Don't put service IPs in documents.

## Progressive Disclosure

Not every tool needs to be fully loaded at session start. Tool definitions can be deferred — the agent sees tool names but not full schemas until it needs them. This saves significant context:

| Approach | Token Cost |
|---|---|
| 200 tools fully loaded | ~15-20K tokens |
| 200 tool names, schemas deferred | ~2K tokens |

The agent requests full schemas when it needs a specific tool. This is a 10x reduction in baseline context cost.

## Context Budgets

Every component that loads into context should have a budget:

| Component | Budget | Enforcement |
|---|---|---|
| Session memory | 4K tokens max | Keep MEMORY.md under 200 lines |
| Skill descriptions | 32K tokens | Environment variable cap |
| Tool definitions | Deferred by default | Load on demand |
| File reads | Read specific sections, not entire files | Use line ranges |
| Search results | Top N results, not all | Limit parameters |

## Staleness and Verification

Context from memory is a claim about the past, not a statement about the present. A memory entry saying "function X is in file Y at line 200" was true when written. It may not be true now.

**Rule:** Before acting on a memory, verify it against current state. If a memory names a file, check the file exists. If it names a function, grep for it. If the user is about to act on your recommendation, verify first.

"The memory says X exists" is not the same as "X exists now."

## Anti-Patterns

- **Loading everything at session start.** Most context is irrelevant to any given session. Load the minimum, fetch the rest on demand.
- **Duplicating information across layers.** If it's in the knowledge graph, don't also put it in session memory. One source of truth per fact.
- **Storing code patterns in memory.** Code changes. Memory doesn't track changes. Read the current code instead of remembering what it used to look like.
- **Unbounded tool results.** A search that returns 500 results burns context on data the agent will never use. Always limit.
