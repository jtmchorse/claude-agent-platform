# Claude Agent Platform

A production multi-agent AI platform built on [Claude Code](https://claude.ai/code) — 44 composable skills, 5 MCP servers exposing 110+ tools, tiered agent orchestration, and a self-improving feedback loop.

This repo showcases the architecture, patterns, and representative skills from a personal infrastructure management platform. All examples are genericized — no API keys, personal paths, or sensitive data.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Code (Parent)                   │
│              Opus model · Decision & synthesis            │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Research  │  │ Briefing │  │  Deploy  │  │  Focus   │ │
│  │  Skill   │  │  Skill   │  │  Skill   │  │  Skill   │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘ │
│       │              │              │              │       │
│  ┌────▼──────────────▼──────────────▼──────────────▼────┐ │
│  │              MCP Tool Layer (110+ tools)              │ │
│  ├──────────┬──────────┬──────────┬──────────┬─────────┤ │
│  │ Homelab  │Remembr.  │   n8n    │ Memory   │Context7 │ │
│  │ 37 tools │ 22 tools │ 20 tools │ 9 tools  │ 2 tools │ │
│  └────┬─────┴────┬─────┴────┬─────┴────┬─────┴────┬────┘ │
│       │          │          │          │          │       │
└───────┼──────────┼──────────┼──────────┼──────────┼───────┘
        ▼          ▼          ▼          ▼          ▼
   Containers   Vault     Workflows   Knowledge   Library
   & Services  (Markdown)  (n8n)       Graph       Docs
```

## Key Concepts

### Composable Skills
Each skill is a self-contained markdown file defining a workflow with clear inputs, outputs, and failure modes. Skills compose through shared external state, not internal coupling.

```
skills/
├── research/        # Multi-engine web research with trust scoring
├── briefing/        # Morning briefing from 6+ data sources
├── end-session/     # Learning capture + state sync
├── retro/           # Monthly friction-to-protocol review
├── learning-extractor/  # Classify learnings into config patches
├── focus/           # Timed work sprints with boundary enforcement
├── deploy/          # Service deployment with health checks
└── ... (44 total)
```

### Tiered Agent Orchestration
- **Parent agent** (high-capability model): Plans, dispatches, synthesizes, decides
- **Subagents** (fast/cheap model): Data gathering, file scanning, web search — parallelized
- 10 parallel research agents return in 60-90s. Parent synthesizes in minutes, not the 15+ of sequential execution.

### Shared State (3 layers)
1. **Knowledge graph** — structured entities and relations (services, decisions, patterns)
2. **Vault** — markdown documents organized by PARA method
3. **Session memory** — conversation-scoped context loaded at start

### Self-Improvement Loop
```
/end-session  →  Capture learnings (what worked, what broke, what to try)
                      ↓
/learning-extractor  →  Classify into: config fixes, new rules,
                        tool preferences, service gotchas, skill candidates
                      ↓
/retro (monthly)  →  Review all learnings, find repeated frictions,
                     propose automations, prune stale data
                      ↓
                  System gets more reliable over time
```

### Operational Principles

These emerged from real failures and are encoded in the platform:

1. **Separate Explore from Execute** — "find" and "suggest" have no side effects. "request" and "deploy" do. Never cross the boundary without confirmation.
2. **Verify Before Batch** — Test 1-2 operations first, inspect results, then batch the rest.
3. **Check State After Mutation** — After any API call that changes state, verify the result.
4. **Know the Layer Below** — Every MCP tool wraps an API. When the tool fails, drop to the raw API.

## Skills in This Repo

| Skill | Purpose | Key Pattern |
|-------|---------|-------------|
| [research](skills/research/) | Multi-engine web research with trust scoring | Parallel subagent dispatch, source deduplication |
| [briefing](skills/briefing/) | Morning briefing from multiple data sources | Parallel MCP tool calls, graceful degradation |
| [end-session](skills/end-session/) | Session close with learning capture | Self-improvement loop entry point |
| [learning-extractor](skills/learning-extractor/) | Classify learnings into system patches | Automated config generation |
| [retro](skills/retro/) | Monthly retrospective | Friction pattern detection, automation proposals |
| [focus](skills/focus/) | Timed work sprints with boundary enforcement | Background timers, escalating notifications |

## What's Not Here

- The full MCP server implementations (proprietary infrastructure)
- API keys, service endpoints, personal data
- The complete skill library (44 skills — 6 representative ones shown)
- The knowledge graph contents

## The Write-Up

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full narrative: how this evolved from a single monolithic prompt to a 44-skill platform, including what broke along the way.

## Built With

- [Claude Code](https://claude.ai/code) — AI coding agent
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) — tool integration standard
- [Proxmox VE](https://www.proxmox.com/) — infrastructure platform
- [n8n](https://n8n.io/) — workflow automation
