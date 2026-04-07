<p align="center">
  <img src="assets/banner.png" alt="Claude Agent Platform" width="800" />
</p>

<h3 align="center">
  From monolithic prompts to a self-improving multi-agent platform
</h3>

<p align="center">
  <img src="https://img.shields.io/badge/skills-44-blue" alt="44 Skills" />
  <img src="https://img.shields.io/badge/MCP_servers-5-purple" alt="5 MCP Servers" />
  <img src="https://img.shields.io/badge/tools-110%2B-green" alt="110+ Tools" />
  <img src="https://img.shields.io/badge/built_with-Claude_Code-orange" alt="Built with Claude Code" />
  <img src="https://img.shields.io/badge/model-Opus_4-red" alt="Opus 4" />
</p>

<p align="center">
  <a href="ARCHITECTURE.md">Full Write-Up</a> ·
  <a href="#demos">Demos</a> ·
  <a href="#skills">Skills</a> ·
  <a href="examples/memory-architecture.md">Memory Architecture</a>
</p>

---

> **Meta**: The company research that inspired this repo was itself conducted by this platform — 4 parallel research agents, 150+ web sources, synthesized in under 3 minutes. The system is the proof.

---

## The Problem

Managing infrastructure, career tracking, research, and daily operations means juggling dozens of tools, APIs, and manual workflows. A single AI prompt can help with one task. But real work is **multi-step, multi-source, and needs to remember what it learned**.

I started with one big prompt. Six months later, it's a platform.

## The Result

| Before | After |
|--------|-------|
| One monolithic prompt doing everything | **44 composable skills**, each with clear purpose |
| Sequential execution, one task at a time | **Parallel subagent dispatch** — 10 agents in 60 seconds |
| Context lost between sessions | **3-layer persistent memory** (knowledge graph + vault + session) |
| Same mistakes repeated | **Self-improvement loop** — system patches itself |
| Manual tool juggling | **110+ MCP tools** across 5 servers |

---

## Demos

### Research: 4 Parallel Agents → Structured Report

<!-- DEMO: /research command dispatching 4 parallel haiku agents searching different angles,
     results streaming back, parent synthesizing, report saved to vault.
     Target: ~60s asciinema recording or GIF
     Show: the agent dispatch messages, parallel execution, synthesis, file write confirmation
     File: assets/demo-research.gif -->

> `/research lago billing platform` → dispatches 4 agents (Practitioner, Skeptic, Economist, Historian) → each searches 2-3 queries → reads top sources → returns structured findings with trust scores → parent deduplicates, flags contradictions, synthesizes → saves to vault

### Morning Briefing: 6 Sources in Parallel

<!-- DEMO: /briefing command fetching weather, meals, calendar, infra health, career, tasks
     all in parallel, formatted output appearing section by section.
     Target: ~30s asciinema recording or GIF
     Show: parallel MCP tool calls firing, data returning, clean formatted output
     File: assets/demo-briefing.gif -->

> `/briefing` → fetches weather, meals, meetings, infrastructure health, career metrics, open tasks — all in parallel → formats into a scannable morning summary

### Self-Improvement: Learning → System Patch

<!-- DEMO: /end-session capturing learnings, then /learning-extractor classifying them
     into config fixes, operational principles, skill candidates.
     Target: ~45s asciinema recording or GIF
     Show: learning bullets written, classification happening, config patch proposed
     File: assets/demo-self-improvement.gif -->

> `/end-session` → captures what worked and what broke → `/learning-extractor` classifies each learning → proposes config patches, new rules, skill candidates → approved changes applied automatically

---

## How It Works

```mermaid
graph TB
    User([fa:fa-user You]) --> CC[fa:fa-brain Claude Code<br/>Parent Agent - Opus]

    CC --> Skills{fa:fa-puzzle-piece Skills Layer<br/>44 composable skills}

    Skills --> R[fa:fa-search /research]
    Skills --> B[fa:fa-sun /briefing]
    Skills --> E[fa:fa-flag /end-session]
    Skills --> F[fa:fa-clock /focus]
    Skills --> D[fa:fa-rocket /deploy]
    Skills --> More[... 39 more]

    R --> |dispatches| Sub1[fa:fa-bolt Subagent 1<br/>Haiku - fast]
    R --> |dispatches| Sub2[fa:fa-bolt Subagent 2<br/>Haiku - fast]
    R --> |dispatches| Sub3[fa:fa-bolt Subagent 3<br/>Haiku - fast]
    R --> |dispatches| Sub4[fa:fa-bolt Subagent 4<br/>Haiku - fast]

    CC --> MCP{fa:fa-plug MCP Tool Layer<br/>110+ tools}

    MCP --> H[Homelab<br/>37 tools]
    MCP --> Rem[Remembrancer<br/>22 tools]
    MCP --> N8[n8n<br/>20 tools]
    MCP --> Mem[Memory<br/>9 tools]
    MCP --> C7[Context7<br/>2 tools]

    H --> Infra[(Containers<br/>& Services)]
    Rem --> Vault[(Vault<br/>Markdown)]
    N8 --> WF[(Workflows)]
    Mem --> KG[(Knowledge<br/>Graph)]

    style CC fill:#f96,stroke:#333,color:#fff
    style Sub1 fill:#6cf,stroke:#333,color:#fff
    style Sub2 fill:#6cf,stroke:#333,color:#fff
    style Sub3 fill:#6cf,stroke:#333,color:#fff
    style Sub4 fill:#6cf,stroke:#333,color:#fff
    style KG fill:#c9f,stroke:#333
    style Vault fill:#9f9,stroke:#333
```

### The Key Insight: Subagents Gather, Parent Decides

The expensive model (Opus) never wastes tokens on web searches or file scanning. It **plans the work, dispatches cheap/fast subagents (Haiku), and synthesizes their results**. This isn't just a performance optimization — it's an architectural boundary that prevents the wrong model from making the wrong decisions.

| Layer | Model | Role | Cost |
|-------|-------|------|------|
| Parent | Opus | Plan, dispatch, synthesize, decide | High |
| Subagents | Haiku | Search, fetch, scan, gather | Low |
| Tools | MCP | Execute operations on external systems | Free |

---

## Shared State: 3 Layers

Multi-agent coordination requires shared context. Three layers, each optimized for different access patterns:

```mermaid
graph LR
    subgraph "Layer 1: Knowledge Graph"
        KG[Structured entities<br/>& relations]
    end

    subgraph "Layer 2: Vault"
        V[Markdown documents<br/>PARA method]
    end

    subgraph "Layer 3: Session Memory"
        SM[MEMORY.md<br/>Conversation-scoped]
    end

    KG -->|"slow to update<br/>fast to query"| V
    V -->|"easy to write<br/>slow to search"| SM
    SM -->|"tiny but<br/>always present"| KG

    style KG fill:#c9f,stroke:#333
    style V fill:#9f9,stroke:#333
    style SM fill:#ff9,stroke:#333
```

| Layer | Best For | Written By | Read By |
|-------|----------|-----------|---------|
| **Knowledge Graph** | Facts that change slowly (services, decisions, patterns) | `/end-session`, `/retro` | All skills at session start |
| **Vault** | Detailed documents (reports, research, daily notes) | `/research`, `/briefing`, subagents | Research prior-art checks, briefings |
| **Session Memory** | Routing decisions, preferences, conventions | `/end-session` | Auto-injected every session |

> See [memory-architecture.md](examples/memory-architecture.md) for the full design.

---

## The Self-Improvement Loop

This is the part that makes it compound. The system learns from every session:

```mermaid
graph LR
    ES["/end-session<br/>Capture learnings"] --> LE["/learning-extractor<br/>Classify & patch"]
    LE --> |"config fixes"| CF[settings.json]
    LE --> |"new rules"| NR[CLAUDE.md]
    LE --> |"tool preferences"| TP[Documentation]
    LE --> |"service gotchas"| SG[Knowledge Graph]
    LE --> |"skill candidates"| SC[New Skills]

    RET["/retro (monthly)<br/>Find repeated friction"] --> |"proposes"| ES

    CF --> Better["System gets<br/>more reliable"]
    NR --> Better
    TP --> Better
    SG --> Better
    SC --> Better

    Better --> |"next session"| ES

    style ES fill:#f96,stroke:#333,color:#fff
    style LE fill:#6cf,stroke:#333,color:#fff
    style RET fill:#c9f,stroke:#333,color:#fff
    style Better fill:#9f9,stroke:#333
```

**Real example**: Research tasks kept re-searching topics already covered. The learning-extractor flagged the pattern. A "check research index before searching" rule was added to CLAUDE.md. Now the system checks what it knows before hitting the web. Automatically. Every time.

**Another example**: Agents kept hitting permission errors writing to the vault. The extractor proposed a standing permission rule. Applied once, never hits that error again.

---

## Operational Principles

These aren't theoretical. Each one came from a real failure:

<details>
<summary><strong>Separate Explore from Execute</strong></summary>

"Find", "suggest", "research" = **explore mode** (no side effects).
"Request", "download", "deploy" = **execute mode** (has side effects).

**Why**: An agent researching "what if we changed the config?" accidentally changed the config. Now there's a hard boundary: explore mode never crosses into execute without explicit human confirmation.

**How it's enforced**: Skills declare their mode. The parent agent checks before dispatching. MCP tools that mutate state require confirmation gates.
</details>

<details>
<summary><strong>Verify Before Batch</strong></summary>

Never fire off N parallel operations assuming they'll all succeed. Test 1-2 first, inspect results, then batch.

**Why**: A batch book request sent 15 API calls. 12 grabbed study guides instead of novels because the search API does fuzzy matching. Now: test 2, verify titles, then batch the rest.

**How it's enforced**: Batch operations in skills have a mandatory "test phase" before "execute phase."
</details>

<details>
<summary><strong>Check State After Mutation</strong></summary>

After any API call that changes state, verify the resulting state. Don't trust "OK" responses.

**Why**: An `addAuthor` API returned 200 OK but silently set the author to "Paused" status. Nothing happened until someone checked. Now: every mutation is followed by a verification query.
</details>

<details>
<summary><strong>Know the Layer Below</strong></summary>

Every MCP tool wraps an API. When the tool returns unexpected results, drop to the raw API.

**Why**: An MCP book search tool grabbed the wrong edition 50% of the time. The underlying API had a `findBook` endpoint with better filtering. Document both paths — the MCP shortcut AND the direct API fallback.
</details>

---

## Skills

6 representative skills from the full library of 44:

| Skill | What It Does | Key Pattern |
|-------|-------------|-------------|
| [/research](skills/research/) | Multi-engine web research across 3 depth modes | Parallel subagent dispatch with structured output contracts and mechanical trust scoring |
| [/briefing](skills/briefing/) | Morning briefing from 6+ parallel data sources | Graceful degradation — one source failing doesn't block the rest |
| [/end-session](skills/end-session/) | Captures learnings and syncs all state | Entry point for the self-improvement loop. Fast (2 min) and full (8 min) modes |
| [/learning-extractor](skills/learning-extractor/) | Classifies learnings into system patches | 6 categories: config fixes, principles, tool preferences, gotchas, skill candidates, noise |
| [/retro](skills/retro/) | Monthly review of all session friction | Finds repeated patterns, proposes automations, prunes stale data, audits decisions |
| [/focus](skills/focus/) | Timed work sprints with escalating boundaries | Background timer → terminal nudge → push notification → forced wrap-up with auto-logging |

<details>
<summary><strong>The other 38 skills</strong></summary>

Deployment, infrastructure auditing, service health checks, container management, backup orchestration, bookmark digestion, book pipeline management, knowledge graph export, daily planning reviews, career tracking, meeting prep, product research, life reviews, service onboarding, configuration auditing, PR review, feature scaffolding, shipping workflows, log viewing, messaging, and more.

Each follows the same pattern: clear purpose, defined inputs/outputs, failure modes documented, composable through shared state.
</details>

---

## What I'd Build Next

Everything here is single-operator. The hardest unsolved problem is **multi-user coordination** — different people with different permissions, different views, different triggers, sharing the same state layer.

The foundation supports it: composable skills, tiered orchestration, external shared state, self-improvement loops. What's missing is the coordination layer that lets a team of humans interact with a team of agents.

That's the problem I want to solve next.

---

## Project Structure

```
claude-agent-platform/
├── README.md                          # You are here
├── ARCHITECTURE.md                    # Full narrative: monolith → platform
├── skills/
│   ├── research/SKILL.md              # Parallel subagents, trust scoring
│   ├── briefing/SKILL.md              # Parallel fetch, graceful degradation
│   ├── end-session/SKILL.md           # Learning capture, self-improvement entry
│   ├── learning-extractor/SKILL.md    # Learning → system patch classification
│   ├── retro/SKILL.md                 # Monthly friction → protocol
│   └── focus/SKILL.md                 # Timed sprints, escalating boundaries
├── examples/
│   ├── CLAUDE.md                      # Genericized operational principles
│   └── memory-architecture.md         # 3-layer shared state design
└── assets/
    ├── banner.png                     # Repo banner
    ├── demo-research.gif              # Research skill demo
    ├── demo-briefing.gif              # Briefing skill demo
    └── demo-self-improvement.gif      # Learning loop demo
```

---

<p align="center">
  <sub>Built with <a href="https://claude.ai/code">Claude Code</a>, 5 MCP servers, and a Supermicro 1U that refuses to quit.</sub>
</p>
