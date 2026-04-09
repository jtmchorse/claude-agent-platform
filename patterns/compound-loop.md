# Pattern: The Compound Loop

A system architecture where each work session improves the next one automatically.

## The Problem

AI-assisted development has a memory problem. Every session starts from zero. Mistakes repeat. Context rebuilds from scratch. The human carries all the learning between sessions, which means the system's effectiveness is capped by human recall.

## The Pattern

Wire three feedback paths into your agent system:

```
Session N
  → Capture learnings (what worked, what broke, what surprised you)
  → Classify each learning (config fix? new rule? skill candidate? noise?)
  → Apply approved changes to system config
  → Next session inherits the improvements automatically

Session N+1 is measurably better than Session N.
```

## Implementation

### Layer 1: Capture

At session end, extract learnings. Not a journal entry. Structured observations:

```markdown
- [tooling] MCP session churn: 15min reconnects are client-side, not server
- [discovery] BillingStatus.NONE + valid stripeCustomerId is intentional pre-checkout state
- [feedback] User wants terse responses with no trailing summaries
```

Each learning has a category tag. The tag determines where it routes.

### Layer 2: Classify

Each learning maps to a system change type:

| Classification | Action | Example |
|---|---|---|
| Config fix | Update settings.json or CLAUDE.md | "Add permission rule for vault writes" |
| Operational principle | Add to CLAUDE.md conventions | "Check research index before web searches" |
| Tool preference | Update tool selection rules | "Use searxng over built-in WebSearch" |
| Service gotcha | Add to knowledge graph | "n8n Code nodes: use this.helpers.httpRequest() not fetch()" |
| Skill candidate | Flag for future build | "Friction pattern X repeated 3 times, automate it" |
| Noise | Discard | Session-specific, not reusable |

### Layer 3: Apply

Approved changes become part of the system:
- CLAUDE.md rules govern agent behavior every session
- settings.json permissions prevent repeated permission prompts
- Knowledge graph entries surface via search when relevant
- New skills automate repeated friction patterns

### Layer 4: Review (Monthly)

The individual session loop catches tactical improvements. A monthly review catches strategic patterns:
- Which friction patterns repeated across 10+ sessions?
- Which skills are never used? (Prune them)
- Which knowledge graph entities are stale? (Update or delete)
- What manual work still happens that should be automated?

## Why It Compounds

Session 1 has zero context, zero rules, zero learned preferences.
Session 50 has 200+ rules, 30 operational principles, 100+ knowledge graph entities, and 40 skills that each represent an automated solution to a problem that used to require manual work.

Session 442 is operating on a fundamentally different surface than Session 1. Not because the model got better. Because the system around it did.

## The Key Constraint

The human reviews every proposed change before it's applied. The system proposes. The human approves. This prevents drift, hallucinated rules, and cargo-cult automation. The compound loop is human-in-the-loop by design, not by limitation.

## Anti-Patterns

- **Capturing everything.** Most session activity is routine. Only capture what's surprising, what broke, or what you'd want to know next time.
- **Skipping the classification step.** Unclassified learnings pile up and never get applied. The classification IS the value.
- **Automating without reviewing.** A system that patches itself without human oversight will drift into nonsense within weeks.
- **Building skills too early.** Wait for a friction pattern to repeat 3+ times before automating. Premature skills rot.
