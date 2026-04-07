---
name: research
description: Multi-engine web research with trust scoring, parallel subagent dispatch, and structured output contracts. Three depth modes.
---

# Research v3.1

Compile web research into a structured reference document. Three depth modes. Multi-engine search. Vault-first design — check what you know, then go to the web for what you don't.

## Step 1: Define the Research

Ask the user:
> "What are you researching? Any starting URLs or specific questions to answer?"

Capture:
- **Topic** — the subject (used for filename and title)
- **Starting URLs** — any links the user already has
- **Key questions** — what specifically they want answered
- **Depth** — ask or infer from context:
  - `shallow` — quick factual lookup, single-agent, < 5 min (default)
  - `standard` — parallel agents, source dedup, contradiction flagging, ~10 min
  - `deep` — perspectival swarm with cognitive lenses + synthesis, ~20 min

## Step 2: Check Prior Research

Before hitting the web, search what we already know. This is not optional.

1. **Vault search** — search prior reports on this topic
2. **Knowledge graph** — search stored findings
3. **Session learnings** — scan for relevant patterns

**Evaluate freshness by topic velocity, not calendar:**

| Topic velocity | Still fresh | Needs update |
|---------------|-------------|-------------|
| Fast-moving (AI, pricing, startup landscape) | < 2 weeks | > 2 weeks |
| Medium (frameworks, tools, industry trends) | < 2 months | > 2 months |
| Slow (protocols, standards, fundamentals) | < 6 months | > 6 months |

If prior research exists and is still fresh: tell the user what you found and ask if they want a refresh.

**If prior research exists but needs updating:** extract prior findings and pass them to research agents as `KNOWN_CONTEXT`. Agents should focus on what's changed, not re-confirm known facts.

## Step 3: Gather Sources

### Shallow Mode (single agent)

Search directly — no subagents. 2-3 queries → read top 3-5 sources → compile.

### Standard Mode (parallel agents)

Dispatch 3-4 **subagents** (fast model) in parallel. Each agent gets a different search angle but uses the same structured output contract.

**Why a reliable model matters here:** The structured output contract is the backbone of synthesis. Fast models can drift from the format, making synthesis interpretive instead of mechanical. Pick a model that follows contracts reliably.

Each agent prompt MUST use this template:

```
You are a research agent investigating: {topic}

Your search angle: {angle_description}
Suggested queries: {query_list}

KNOWN_CONTEXT (do not re-confirm, focus on NEW information):
{prior_findings_summary_or_"None"}

Instructions:
1. Run 2-3 search queries
2. Read the top 3-5 most relevant results
3. Return findings in the EXACT format below — do NOT write files

## FINDINGS

### [Finding Title]
- **Claim:** [Specific factual claim — not a summary, a testable statement]
- **Source:** [Full URL]
- **Source date:** [Publication date if visible, or "undated"]
- **Source type:** official_docs | org_blog | tech_blog | forum | news | academic | personal_blog | unknown
- **Corroborated by:** [URL of second source confirming this, or "uncorroborated"]

[Repeat for each finding, aim for 4-8]

## META
- Queries used: [list each search query]
- URLs consulted: [N]
- URLs used in findings: [N]
- Gaps: [What you looked for but couldn't find]
```

After all agents return:
1. **Deduplicate** — same source URL across agents → keep more detailed entry
2. **Score trust** mechanically (see Trust Scoring below)
3. **Flag contradictions** — where agents found conflicting claims
4. **Check for iteration** — if agents collectively identified gaps that matter, dispatch 1-2 targeted follow-ups

### Deep Mode (perspectival swarm)

Dispatch 4 persona agents, each with a cognitive lens:

| Persona | Focus | Query style |
|---------|-------|-------------|
| **The Practitioner** | Who's doing this in production? War stories? | `"{topic} production experience"`, `"{topic} lessons learned"` |
| **The Skeptic** | What's failing? What's overhyped? | `"{topic} problems"`, `"migrating away from {topic}"` |
| **The Economist** | Who profits? What are the incentives? | `"{topic} pricing"`, `"{topic} business model"` |
| **The Historian** | What came before? What cycle is this repeating? | `"{topic} history"`, `"before {topic}"` |

After all return, synthesize:
1. Deduplicate across personas
2. Map agreements (claims found by 2+ personas = high confidence)
3. Flag contradictions with both sides
4. Write Blind Spots: topics absent from ALL results, incentive analysis, historical precedent, failure patterns

## Trust Scoring

Trust is computed per-finding, not per-source. A finding's trust level is the **lowest** of its three factors:

| Factor | High | Medium | Low |
|--------|------|--------|-----|
| **Source type** | official_docs, academic, org_blog | tech_blog, news | personal_blog, forum, unknown |
| **Recency** | Within topic's "fresh" window | Within 2x fresh window | Older than 2x |
| **Corroboration** | Corroborated by 1+ independent source | Uncorroborated from high authority | Uncorroborated from medium/low |

This is mechanical: `source_type=forum` + `corroborated=no` = `low` regardless of how convincing the claim sounds.

## Iteration Loop

After initial results:
1. **Same gap flagged by 2+ agents?** → Dispatch targeted follow-up
2. **Findings invalidate original framing?** → Pause, tell the user
3. **High-value URL nobody read fully?** → Deep-read it

**Limit:** Max 2 follow-up agents. More than that means the scope was wrong.

## Step 4: Compile & Save

Use structured templates (shallow/standard/deep) with:
- Specific details (prices, dates, URLs)
- Flagged uncertainty ("appears to be" vs "confirmed")
- Actionable next steps
- Source attribution table with trust scores
- Low-trust findings in appendix only

## Step 5: Report

```
Research saved:
- Depth: {shallow|standard|deep}
- Agents dispatched: {N}
- Sources: {N} consulted, {N} used
- Trust: {N} high, {N} medium, {N} low
- Contradictions: {N} flagged
- Follow-ups: {N} dispatched
```
