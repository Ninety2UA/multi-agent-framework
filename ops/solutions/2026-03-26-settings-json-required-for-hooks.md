---
title: "Hooks must be registered — originally in settings.json, now in plugin hooks.json"
date: 2026-03-26
updated: 2026-03-31
tags: [hooks, settings, configuration, claude-code, plugin]
agent: Claude Code (Opus)
---

## Problem
All 3 lifecycle hooks (session-start.sh, ship-loop.sh, context-monitor.sh) existed as executable scripts but never fired during sessions. The framework appeared non-functional despite all files being present and correct.

## Root cause
Claude Code hooks must be **registered** in a configuration file. Without registration, Claude Code has no way to discover hook scripts regardless of their location or permissions.

## Solution

**v2.0.0 (plugin):** Hooks are registered in `hooks/hooks.json` using `${CLAUDE_PLUGIN_ROOT}` paths. This is handled automatically by the plugin system — no manual configuration needed.

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/handlers/session-start.sh" }]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/handlers/ship-loop.sh" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/handlers/context-monitor.sh" }]
      }
    ]
  }
}
```

**Important:** The flat format `{ "command": "...", "timeout": 5000 }` does NOT work. Each entry must have `matcher` (string — tool name, pipe-separated list, or `""` to match all) and `hooks` (array of `{ "type": "command", "command": "..." }` objects). Claude Code will skip the entire config if this format is wrong.

## Prevention
- Plugin users get hooks automatically — no manual setup
- For project-level hooks, use the correct matcher/hooks array format
- Verify hooks fire by running `/status` (session-start hook should produce orientation message)
- Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths in hook commands
