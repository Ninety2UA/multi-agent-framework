---
problem: "Hook scripts read environment variables ($CLAUDE_TOOL_NAME, $CLAUDE_STOP_ASSISTANT_MESSAGE) that Claude Code does not set — making completion detection and tool classification completely non-functional"
context: "ship-loop.sh (Stop hook), context-monitor.sh (PostToolUse hook)"
solution: "Parse stdin JSON with python3 instead of reading env vars"
date: 2026-03-31
agent: Claude Code
---

## Problem

Both `ship-loop.sh` and `context-monitor.sh` read hook-specific data from environment variables:
- `$CLAUDE_STOP_ASSISTANT_MESSAGE` in ship-loop.sh (line 23)
- `$CLAUDE_TOOL_NAME` in context-monitor.sh (line 10)

Claude Code does NOT set these environment variables. Hook data is delivered as JSON on stdin.

**Impact:** ship-loop completion detection never fired (every sprint burned all 5 iterations). Context-monitor tool classification always fell to "unknown" (paralysis detection never triggered).

## Root cause

The hook scripts were written assuming an environment-variable-based hook interface. Claude Code's actual hook interface delivers structured JSON on stdin with fields like `tool_name`, `last_assistant_message`, `hook_event_name`, etc.

## Solution

Replace environment variable reads with stdin JSON parsing using python3 (available on macOS by default):

```bash
# Stop hook — read last_assistant_message
HOOK_INPUT=$(cat)
LAST_MESSAGE=$(echo "$HOOK_INPUT" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('last_assistant_message',''))" 2>/dev/null || echo "")

# PostToolUse hook — read tool_name
HOOK_INPUT=$(cat)
TOOL_NAME=$(echo "$HOOK_INPUT" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_name','unknown'))" 2>/dev/null || echo "unknown")
```

Key points:
- `cat` reads all stdin before any processing (stdin is consumed once)
- `2>/dev/null || echo "fallback"` handles parse failures gracefully
- python3 is available on macOS by default (no extra dependencies)

## Prevention

- Always check Claude Code hook documentation for the actual data delivery mechanism before writing hooks
- Test hooks by adding `echo "$HOOK_INPUT" > /tmp/hook_debug.txt` to inspect actual input
- Never assume environment variables exist for hook-specific data — verify first
