---
name: focus
description: Timed work sprints with escalating boundary enforcement — terminal nudges, push notifications, automatic wrap-up and logging.
---

# Focus

Timed work sprints with escalating exit pressure. Not about productivity — about boundaries.

## Arguments

`/focus <duration>m [task description] [break:<minutes>m]`

Examples:
- `/focus 30m refactor auth middleware`
- `/focus 45m TICKET-305`
- `/focus 25m break:10m fix the thing`

## Step 1: Parse Arguments

Extract:
- `DURATION` — minutes (required)
- `TASK` — description (optional, default: "focused work")
- `BREAK_MIN` — from `break:Xm` (optional)

## Step 2: Pick Escalation Messages

Load from message bank. Three tiers:
- **Tier 1** (nudge) — gentle terminal reminder when time's up
- **Tier 2** (escalate) — push notification at duration + 5min
- **Tier 3** (nag) — second push at duration + 15min

## Step 3: Start Timer

Launch background timer process that:
1. At `DURATION`: writes nudge marker file
2. At `DURATION + 5m`: sends Tier 2 push notification
3. At `DURATION + 15m`: sends Tier 3 push notification

Write state to `/tmp/focus-session.json`:
```json
{
  "start_epoch": 1234567890,
  "duration_min": 30,
  "task": "refactor auth",
  "pid": 12345
}
```

## Step 4: Confirm & Go Silent

```
Focus: 30m — refactor auth. Go.
```

Then **stop talking**. The user is working.

## Step 5: Nudge Check

On any subsequent user message, check for nudge marker. If present:
1. Display Tier 1 message
2. Ask: "Wrap up now, or keep going?"

## Step 6: Wrap Up

When the user finishes (explicitly or via nudge):

1. Kill timer process
2. Calculate elapsed time
3. Jot to daily notes: `## [HH:MM] — [focus] ${TASK} — ${ELAPSED}m / ${PLANNED}m`
4. Auto-commit if working tree is dirty
5. Sync task tracker

If break was requested, start break timer with push notification callback.

```
Get away from the desk. The code will be here when you get back.
```

## Design Decisions

- **Escalating pressure, not blocking**: The system nudges, then pushes, then nags — but never stops you. You're an adult.
- **Automatic logging**: If you don't log it, it didn't happen. The jot is automatic.
- **Break timer**: Physical separation from the desk is the point. Push notification brings you back.
- **State file, not memory**: Timer state lives in `/tmp/`, not in conversation context. Survives context compaction.
