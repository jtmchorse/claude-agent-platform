# Pattern: Parallel Agent Dispatch with Tiered Model Selection

How to orchestrate multiple AI agents for research, data gathering, and synthesis without burning expensive tokens on cheap work.

## The Problem

A single agent doing research sequentially is slow and expensive. It searches, reads, searches again, reads again. Each step burns high-capability model tokens on work that doesn't require high capability.

## The Pattern

Separate planning/synthesis from gathering. Use different models for different roles.

```
Parent (expensive model)
  ├── defines research angles
  ├── dispatches N subagents in parallel
  │     ├── Subagent 1 (cheap model): searches angle A, reads top 3 results
  │     ├── Subagent 2 (cheap model): searches angle B, reads top 3 results
  │     ├── Subagent 3 (cheap model): searches angle C, reads top 3 results
  │     └── Subagent 4 (cheap model): searches angle D, reads top 3 results
  ├── receives all results simultaneously
  ├── deduplicates, flags contradictions
  └── synthesizes into structured output
```

## Why This Works

| Role | Model | Why |
|---|---|---|
| Planning | Expensive | Deciding what to search requires judgment |
| Gathering | Cheap | Web searches, file reads, API calls are mechanical |
| Synthesis | Expensive | Resolving contradictions, ranking sources, forming conclusions requires judgment |

The expensive model never wastes tokens on `fetch(url)`. The cheap model never makes judgment calls about what the data means.

## Implementation Notes

### Subagent Prompts

Each subagent gets a focused brief:
- What angle to research (specific, not vague)
- How many sources to gather (bounded)
- What format to return (structured, not prose)
- What NOT to do (no synthesis, no recommendations, just data)

### Output Contracts

Subagents return structured data, not essays:

```markdown
## Source: [title](url)
**Trust**: high|medium|low (based on domain authority, recency, specificity)
**Key findings**:
- Finding 1
- Finding 2
**Relevance to query**: 1-2 sentences
```

The parent can process structured returns faster than free-form text.

### Failure Handling

Some subagents will fail (bad search results, unreachable URLs, rate limits). The parent handles this gracefully:
- If 3/4 subagents return: synthesize from what you have
- If 1/4 return: report the gap, don't fabricate
- If 0/4 return: report failure, suggest alternative approaches

### Depth Caps

Prevent runaway subagent trees. A subagent should never spawn its own subagents beyond a configurable depth (typically max 2 levels). This prevents exponential resource consumption and keeps the parent in control.

## Cost Comparison

| Approach | Time | Token Cost |
|---|---|---|
| Single expensive model, sequential | 15 min | High |
| 4 cheap subagents parallel + expensive synthesis | 2 min | ~40% of sequential |

The time savings are dramatic. The cost savings are meaningful at volume.

## When NOT to Use This

- Simple lookups that don't need multiple angles
- Tasks that require iterative reasoning (each step depends on the previous)
- Anything where the gathering step requires judgment (use the expensive model directly)
