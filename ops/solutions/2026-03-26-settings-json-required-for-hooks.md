---
title: "Hooks won't fire without .claude/settings.json"
date: 2026-03-26
tags: [hooks, settings, configuration, claude-code]
agent: Claude Code (Opus)
---

## Problem
All 3 lifecycle hooks (session-start.sh, ship-loop.sh, context-monitor.sh) existed as executable scripts but never fired during sessions. The framework appeared non-functional despite all files being present and correct.

## Root cause
Claude Code hooks must be **registered** in `.claude/settings.json` under the `hooks` key. Without this file, Claude Code has no way to discover hook scripts regardless of their location or permissions.

## Solution
Created `.claude/settings.json` with all 3 hooks registered:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "hooks": {
    "SessionStart": [{ "command": ".claude/hooks/session-start.sh", "timeout": 5000 }],
    "Stop": [{ "command": ".claude/hooks/ship-loop.sh", "timeout": 10000 }],
    "PostToolUse": [{ "command": ".claude/hooks/context-monitor.sh", "timeout": 5000 }]
  }
}
```

## Prevention
- Always include `.claude/settings.json` when distributing hooks
- Document required settings in README installation section
- Verify hooks fire after cloning by running `/status` (session-start hook should produce orientation message)
