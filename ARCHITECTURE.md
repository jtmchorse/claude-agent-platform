# From Monolithic Prompts to a 44-Skill Agent Platform: What I Learned Building AI Infrastructure for Myself

## The Setup

I run a homelab — a Supermicro 1U server with Proxmox, 14 containers, and a growing stack of services. About six months ago I started using Claude Code to manage it. What started as "run some commands on containers" turned into something much more interesting: a production-grade multi-agent platform with 44 composable skills, 5 MCP servers exposing 110+ tools, and a self-improving feedback loop that makes the system smarter every session.

I'm not a startup. I'm one person managing home infrastructure, a family, and a dozen side projects. But the engineering problems I solved are the same ones any team building internal AI agents will face: how do you decompose monolithic prompts? How do you orchestrate multiple agents? How do you build systems that learn from their own mistakes?

Here's what I built, what broke, and what I'd do differently.

## Phase 1: The Monolith

It started the way it always does — one big prompt doing everything. Research a topic, fetch web results, parse them, write a report, save it to the right folder, update the index. A single mega-skill that handled enrichment, synthesis, file management, and notification in one pass.

It worked. Until it didn't. The prompt got too long. Context windows filled up. Edge cases multiplied. Adding a new capability meant touching the entire thing. Sound familiar?

## Phase 2: Decomposition

I broke the monolith into discrete, composable skills — each with a clear purpose, defined inputs/outputs, and its own failure modes:

- `/briefing` — morning briefing assembling weather, meals, calendar, infrastructure health, and open tasks
- `/research` — dispatches parallel search agents, gathers sources, synthesizes findings
- `/cartography` — daily planning review with structured priority walkthrough
- `/end-session` — captures learnings, syncs state, ensures nothing is left uncommitted
- `/retro` — monthly review of session learnings, proposes new skills and memory entries
- `/learning-extractor` — classifies session learnings into actionable config patches

Each skill is a self-contained markdown file with execution instructions. No shared mutable state inside the skill. Coordination happens through external systems.

Today there are 44 of these. New ones take hours to build, not days, because the patterns are established.

## Phase 3: Multi-Agent Orchestration

Single-threaded execution was the next bottleneck. A research task that dispatches 10 web searches sequentially wastes minutes. The fix: **parallel subagent dispatch with tiered model selection.**

The pattern:
- **Parent agent** (high-capability model): Plans the work, dispatches subagents, synthesizes results, makes decisions
- **Subagents** (fast/cheap model): Data gathering, file scanning, web fetching, status checks — all parallelizable

A research task now dispatches 10 parallel subagents, each searching a different angle. They all return in 60-90 seconds. The parent synthesizes. Total wall time: ~2 minutes for what used to take 15.

This isn't just a performance optimization — it's an architectural principle. **Subagents gather. The parent decides.** This separation prevents the expensive model from burning tokens on search/fetch work, and prevents the cheap model from making judgment calls it shouldn't.

## Phase 4: The Tool Layer (MCP Servers)

Skills need tools. I built 5 MCP servers exposing 110+ tools:

| Server | Tools | Purpose |
|--------|-------|---------|
| homelab | 37 | Container ops, service health, media requests, search, vault access, Gmail, calendar |
| remembrancer | 22 | Knowledge management, planning, project tracking, dashboard |
| n8n | 20 | Workflow automation management |
| memory | 9 | Knowledge graph CRUD |
| context7 | 2 | Library documentation lookup |

The homelab MCP server (~2,300 lines of JavaScript) wraps SSH, APIs, webhooks, and file operations into clean tool interfaces. When a skill says "check infrastructure health," it calls `homelab_health` — not `ssh ct100 systemctl status...`.

**The key insight: MCP tools are the permission boundary.** Each tool defines what the agent can and cannot do. The agent doesn't get raw shell access to production systems — it gets purpose-built tools with guardrails. This mirrors what any team building internal agents needs: controlled access to sensitive systems through well-defined interfaces.

## Phase 5: Shared State

Multiple agents need shared context. I use three layers:

1. **Knowledge graph** (persistent, structured): Entities and relations — services, patterns, decisions, people. Agents read from it; the end-of-session process updates it.
2. **Vault** (persistent, documents): Markdown files organized by PARA method. Reports, research, project docs, daily notes. Agents can read and write directly.
3. **Session memory** (conversation-scoped): MEMORY.md loaded at start, updated as needed. Keeps the current session oriented without loading everything.

This is the layer that makes multi-agent coordination work. When a research agent writes findings to the vault, a briefing agent can surface them the next morning. When an end-session agent captures a learning, the retro agent can propose a system change a month later.

## Phase 6: The Self-Improvement Loop

This is the part I'm most proud of. The system improves itself.

**The loop:**
1. `/end-session` runs at the end of every work session — captures what was learned, what broke, what worked
2. `/learning-extractor` classifies those learnings into categories: new CLAUDE.md rules, settings changes, new skill ideas, knowledge graph updates
3. `/retro` runs monthly — reviews all session learnings, identifies repeated frictions, proposes new skills or memory entries, prunes stale data

**Example**: I kept hitting permission errors when agents tried to write to the vault. The learning-extractor flagged it. The retro proposed a standing permission rule. Now it's in the config and agents never hit that error again.

**Another example**: Research tasks kept re-searching topics I'd already covered. The system now checks a research index before dispatching web searches. The index is updated automatically when new research is saved.

These aren't manually written corrections — they're systematic. The feedback signal is captured, a change is proposed, a human reviews it, and it becomes part of the system. Over time, the platform gets more reliable, more efficient, and more useful.

## What Broke (Honestly)

**Scope creep through tools.** I started with 20 MCP tools. Now there's 110+. Every tool adds cognitive load for the agent — more options means more potential for wrong choices. I've started being more disciplined about what gets a tool versus what stays as a raw command.

**Prompt engineering is never done.** The CLAUDE.md file that governs agent behavior has been rewritten dozens of times. Early versions were too permissive (agents would push to git without asking). Later versions were too restrictive (agents would ask permission for everything). The current version encodes hard-won principles like "Separate Explore from Execute" — never cross from read-only research into state-changing actions without explicit confirmation.

**Context window management is real.** With 110+ tools loaded, you're already using significant context just for tool definitions. Skills need to be focused. Subagent prompts need to be precise. You learn to be economical with what goes into each agent's context.

## Principles That Emerged

1. **Separate Explore from Execute.** "Find" and "suggest" have no side effects. "Request" and "deploy" do. Never cross the boundary without confirmation. (Some teams call this "Constrain, Confirm, Exclude" — same instinct, different frame.)

2. **Verify Before Batch.** Never fire off N parallel operations assuming they'll all succeed. Test 1-2 first, inspect results, then batch the rest.

3. **Check State After Mutation.** After any API call that changes state, verify the result. Don't trust "OK" responses.

4. **Know the Layer Below.** Every MCP tool wraps an underlying API. When the tool returns unexpected results, drop to the raw API. Document both.

5. **Write-Then-Execute.** When a script exceeds single-command complexity, write it to disk first, then execute. Don't fight quoting limits.

## What This Means for Teams

Everything I built is single-operator. The hardest unsolved problem is making it multi-user — different people with different permissions, different views, different triggers, sharing the same state layer.

That's an engineering problem I'd love to solve next. The foundation is solid: composable skills, tiered agent orchestration, external shared state, and a self-improvement loop. What's missing is the coordination layer that lets a team of humans interact with a team of agents.

---

*Built with Claude Code, 5 MCP servers, and a Supermicro 1U that refuses to quit.*
