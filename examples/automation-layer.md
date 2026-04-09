# The Automation Layer: 60+ Workflows Behind the Skills

Claude Code skills handle interactive work. Behind them: **60+ automated workflows** executing on a schedule, triggered by webhooks, or chained together on a self-hosted n8n instance.

This layer handles everything that should happen without a human in the chair.

## Architecture

```
┌─────────────────────────────────────────┐
│  Claude Code (Interactive)               │
│  Skills, subagents, MCP tools            │
│  Human-in-the-loop, session-based        │
└──────────────┬──────────────────────────┘
               │ triggers / reads from
┌──────────────▼──────────────────────────┐
│  n8n (Automated)                         │
│  60+ workflows, webhook + cron triggers  │
│  Runs 24/7, no human required            │
├─────────────────────────────────────────┤
│  Categories:                             │
│  - Morning briefing (7 sub-workflows)    │
│  - Event classification (5 workflows)    │
│  - Communication bridges (5 workflows)   │
│  - Intelligence digests (7 workflows)    │
│  - Monitoring (6 workflows)              │
│  - Infrastructure (5 workflows)          │
│  - Knowledge/content (8 workflows)       │
│  - Other (17+ workflows)                 │
└─────────────────────────────────────────┘
```

## Key Workflow Families

### Morning Briefing System

One parent workflow dispatches 6 sub-workflows in parallel, each fetching from a different source:

| Sub-workflow | Source | What It Returns |
|---|---|---|
| Weather | Weather API | Forecast, alerts |
| Meals | Meal planning service | Today's plan |
| Meetings | Google Calendar | Today's schedule |
| Tasks | Task tracker + vault | Open items, priorities |
| Projects | Project tracking tools | Active work, blockers |
| Health | Fitness API | Training load, metrics |

All 6 run simultaneously. Results merge into a formatted briefing delivered via push notification. Total execution: ~30 seconds.

### Event Classification Pipeline

Service events and webhook payloads get auto-classified and routed:

```
Container events ───────┐
Webhook payloads ───────┤
Service health checks ──┼──→ Classifier ──→ Structured log
Cron job results ───────┤                    (categorized by type)
API callbacks ──────────┘
```

The classifier tags each event by type and severity, appends it to a structured log, and routes high-priority events to push notifications. A weekly extraction summarizes patterns.

### Intelligence Digests

Automated synthesis of information streams:

- **Dev + AI Daily Digest**: Aggregates AI/dev news from curated sources
- **Weekly Meeting Digest**: Summarizes all meetings into key decisions and action items
- **Notification Intelligence**: Routes and prioritizes notifications by urgency

### Monitoring Workflows

Threshold-based alerts and periodic health checks:

- **System Health**: Service availability, container status, disk usage
- **Resource Alerts**: CPU, memory, and storage thresholds across containers
- **Weekend Brief**: Friday synthesis of the week + upcoming maintenance windows

## The Connection Between Layers

The n8n automation layer and the Claude Code skill layer share state through:

1. **Vault**: Both read and write markdown files to the same document store
2. **Knowledge graph**: Both query the same entity database
3. **MCP tools**: Claude Code skills can trigger n8n workflows via webhook, and n8n can read Claude Code session data

This means a signal captured by an automated workflow at 3am shows up in the morning briefing at 7am, which surfaces in the Claude Code session at 9am. No manual transfer. No copy-paste. The system moves information to where it's needed.

## Why Two Layers

| | Claude Code Skills | n8n Workflows |
|---|---|---|
| **Trigger** | Human prompt | Schedule, webhook, event |
| **Interaction** | Conversational, iterative | Fire and forget |
| **Best for** | Judgment calls, synthesis, creative work | Data movement, scheduling, notification |
| **Model cost** | High (Opus for parent, Haiku for subagents) | Low (API calls, no LLM unless scoring) |
| **Runs** | When you're working | 24/7 |

Some work requires an AI to think. Most work just requires data to move from A to B on a schedule. Using an expensive AI model for the second type is waste. The automation layer handles the mechanical work so the skill layer can focus on the work that actually requires intelligence.
