# Pattern: Session Hooks as Behavioral Guardrails

Using lifecycle hooks to enforce safety, capture data, and inject context automatically — without relying on the AI to remember to do it.

## The Problem

AI agents forget rules between sessions. You can write "never push to production without asking" in your system prompt, but the agent will eventually do it anyway because rules in prompts are suggestions, not enforcement.

## The Pattern

Wire shell scripts into agent lifecycle events. The scripts run OUTSIDE the AI's control — they can block actions, inject context, and capture data regardless of what the agent decides to do.

```
SessionStart
  → Load recent learnings into context
  → Prime task tracking system
  → Inject relevant project state

PreToolUse (before every tool call)
  → Block dangerous commands (rm -rf, force push, etc.)
  → Gate external writes behind approval workflow
  → Track API reads for progressive unlock

PostToolUse (after every tool call)
  → Auto-format edited files
  → Track file access patterns
  → Sync state to external systems

Stop (session end)
  → Verify work was completed properly
  → Write structured trace data
  → Notify user session is done
```

## Implementation: The Approval Gate

The most valuable hook pattern: "make it so" approval for external writes.

```
1. Agent wants to post a Slack message
2. PreToolUse hook intercepts, blocks with exit code 2
3. Agent shows the user exactly what it wants to send
4. User says "make it so"
5. Agent writes a flag file with a 2-minute TTL
6. Agent retries the tool call
7. Hook sees the flag file, approves, deletes the flag
```

This pattern applies to any external write: GitHub PR comments, Jira updates, email drafts, Slack messages. The agent can prepare the content. A human approves the send.

**Why a flag file and not a prompt?** Because the hook runs outside the AI's context. It can't be sweet-talked, prompt-injected, or forgotten. The enforcement is mechanical, not conversational.

## Implementation: Progressive Unlock

Some tools should auto-approve after the agent has demonstrated familiarity:

```
1. First 5 GitHub API reads: require manual approval
2. After 5 reads: non-destructive writes auto-approve (comments, reviews)
3. Destructive writes (merge, close, delete): always require approval
```

The hook tracks read count per service per session. After a threshold, it shifts from blocking to advisory. This reduces friction for the agent while maintaining safety for high-risk operations.

## Implementation: Context Injection

SessionStart hooks can inject context the agent needs but might not think to load:

```bash
# Load last 3 sessions of learnings
cat ~/.claude/shared-memory/session-learnings.md

# Prime the task tracker with available work
bd prime

# Inject any hot-thread context (active incidents, blockers)
cat /tmp/claude-hot-thread-*.md 2>/dev/null
```

The agent starts every session with relevant context without needing to be told to look for it.

## Hook Event Reference

| Event | Fires When | Common Uses |
|---|---|---|
| SessionStart | New session begins | Context injection, task priming |
| PreToolUse | Before any tool call | Safety gates, approval workflows |
| PostToolUse | After any tool call | Formatting, tracking, sync |
| PreCompact | Before context compression | Save state that would be lost |
| UserPromptSubmit | User sends a message | Skill suggestions, input validation |
| Stop | Session ending | Verification, trace writing, notifications |

## Anti-Patterns

- **Hooks that are too slow.** Every hook adds latency to every tool call. Keep them under 500ms. Write async when possible.
- **Hooks that block too aggressively.** If every tool call requires approval, the agent becomes unusable. Gate only external writes and destructive operations.
- **Hooks that duplicate agent behavior.** If the agent already formats code, don't also format in a hook. Pick one owner.
- **Hooks that fail silently.** A broken hook should log the error and allow the operation, not silently block work. Fail open for non-safety hooks, fail closed for safety hooks.
