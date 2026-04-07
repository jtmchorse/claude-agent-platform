---
name: retro
description: Monthly retrospective — reviews session learnings, identifies repeated frictions, proposes automations, prunes stale data. The compound loop that turns friction into protocol.
---

# Retro

Monthly retrospective that turns repeated friction into automated protocol.

**Run:** First of each month, or whenever the system isn't keeping up.

## Step 1: Gather Data

Read from the memory directory:
- `session-learnings.md` — all session entries
- `patterns.md` — current stable patterns
- `decisions.md` — architecture decisions

Also query knowledge graph for system state and check recent git activity.

## Step 2: Knowledge Graph Staleness Check

For each tracked entity, check when it was last validated vs. how much has changed since. Flag entities that have drifted significantly.

## Step 3: Identify Friction Patterns

Scan session learnings for:

1. **Repeated frictions** — same problem appearing 2+ times across sessions
2. **Repeated workflows** — same manual sequence done more than once
3. **Missing patterns** — things that should be codified but aren't
4. **Stale patterns** — things codified that are no longer relevant

Present findings with the sessions where each appeared.

## Step 4: Propose Automations

For each repeated friction or workflow, propose:
- **New skill** — if complex enough for `/skill-name`
- **New pattern entry** — if a simple rule
- **New decision entry** — if an unrecorded architectural choice
- **Memory update** — if a new gotcha or fact

Present as a checklist for user approval.

## Step 5: Prune & Clean

- **Session learnings older than 60 days:** Archive or delete if captured elsewhere
- **Stale patterns:** Remove with commit message explanation
- **Stale knowledge graph entries:** Flag for review (don't delete without asking)

## Step 6: Decision Audit

Review all recorded decisions. For each:
> "Is this decision still valid? Has anything changed?"

Update or remove stale decisions. This keeps the knowledge graph honest.

## Step 7: Velocity Check

Count for the past month:
```
Monthly velocity (YYYY-MM):
- Sessions: [N]
- Patterns: [N] promoted, [N] total
- Decisions: [N] recorded
- Skills: [N] created/modified
```

Not for pressure — for seeing the compound curve.

## Step 8: Execute & Report

Apply approved changes, then:

```
Retro complete:
- Frictions identified: [N]
- Automations proposed: [N]
- Approved & built: [N]
- Entries pruned: [N]
- Next retro: [first of next month]
```
