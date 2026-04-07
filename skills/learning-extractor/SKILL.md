---
name: learning-extractor
user-invocable: false
description: Classify session learnings into actionable system patches — config fixes, operational principles, tool preferences, knowledge graph updates, skill candidates.
---

# Learning Extractor

Transform raw session learnings into concrete system improvements. Runs as part of `/end-session --full` or standalone.

## Step 1: Gather Raw Learnings

Read the latest session learnings from the memory directory. Focus on entries from the last 3 sessions.

## Step 2: Classify Each Learning

For each learning bullet, classify into exactly one category:

### Category A: Config Fix
**Signals:** "had to approve", "permission prompt", "was blocked"
**Action:** Generate the exact JSON patch for settings

### Category B: Operational Principle
**Signals:** "always", "never", "every time", "pattern", appears 2+ sessions
**Action:** Draft principle for project instructions

### Category C: Tool Preference
**Signals:** "use X instead of Y", "X doesn't work for this"
**Action:** Draft tool preference rule

### Category D: Service Gotcha
**Signals:** "API returns", "workaround", specific service name
**Action:** Add observation to knowledge graph entity

### Category E: Skill Candidate
**Signals:** "had to manually", "every time I", "sequence of steps"
**Action:** Draft skill skeleton

### Category F: Noise
One-time fixes, already-resolved issues, too specific to generalize.
**Action:** Skip with reason.

## Step 3: Abstraction Ladder

For each non-F learning:
> "What broader class of problem is this an instance of?"

```
| Specific friction | Abstract class | Where else could this bite us? | Preventive fix |
```

- If a preventive fix exists, verify it works
- If the class appeared in 2+ sessions, flag `[SYSTEMIC]` — create a tracking issue

## Step 4: Present for Approval

```
=== Learning Extractor Results ===

Analyzed: N learnings from last M sessions

CATEGORY A — Config Fixes (N items):
1. [learning] → [specific patch]

CATEGORY B — Operational Principles (N items):
1. [learning] → New principle: "[name]"

CATEGORY C — Tool Preferences (N items):
1. [learning] → Prefer [X] over [Y]

CATEGORY D — Service Gotchas (N items):
1. [learning] → Add observation to [entity]

CATEGORY E — Skill Candidates (N items):
1. [learning] → Draft skill: /[name]

CATEGORY F — No Action (N items):
1. [learning] — [reason skipped]

Ready to apply? (all / pick categories / skip)
```

## Step 5: Apply Approved Changes

Apply each approved category to the appropriate system file or knowledge graph.

## Step 6: Mark Processed

Tag processed learnings with `[EXTRACTED]` so they're not re-processed.
