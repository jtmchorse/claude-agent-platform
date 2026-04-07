---
name: briefing
description: Morning briefing assembling data from 6+ parallel sources — weather, meals, calendar, infrastructure health, career metrics, and open tasks.
---

# Briefing

On-demand morning briefing. Pulls all data sources in parallel and presents a clean summary.

## Arguments

- `--push` (optional): Also send to push notification service
- `--raw` (optional): Show raw JSON from each source

## Step 1: Fetch Data Sources

Call all data sources using **parallel MCP tool calls**:

```
briefing_weather()
briefing_health()
briefing_meals()
briefing_meetings()
briefing_tasks()
briefing_career()
```

Each source is independent — if one fails, show the rest with a note.

## Step 2: Format and Display

```
## Briefing — [Day], [Month] [Date]

**Weather**: [temp]F, [description]. High [high]F, low [low]F.
[If precip > 20%]: Rain chance: [precip]%

**Meetings**: [count] meeting(s)
  Themes: [themes]
  Actions: [top 3 action items]

**Meals**:
  Dinner: [dinner]
  Lunch: [lunch]

**Infrastructure**: [healthy]/[total] services healthy
[If any down]: Down: [service names]

**Career**: [learning progress]
[If 1:1 within 2 days]: 1:1 in [n] day(s) — prep [ready/NEEDED]

**Open Tasks** ([total]):
- [task 1]
- [task 2]
- [task 3]
```

## Step 3: Offer Follow-ups

> "Anything to dig into? I can show details for any section."

## Design Decisions

- **Parallel fetch**: All 6 sources are independent. Fetching sequentially wastes 5-10 seconds.
- **Graceful degradation**: One source failing doesn't block the briefing. Show what you have.
- **Push notifications**: Optional — not everyone wants a phone buzz at 6am.
- **No caching**: Data is always live. A briefing with stale weather is worse than no briefing.
