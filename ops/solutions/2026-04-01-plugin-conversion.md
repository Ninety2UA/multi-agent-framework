---
problem: "Framework installed via git clone + manual file copy was fragile, hard to update, and didn't leverage Claude Code's plugin system"
context: "Converting 49 components (18 agents, 12 skills, 16 commands, 3 hooks) from .claude/ layout to Claude Code plugin layout"
solution: "Restructured as a Claude Code plugin with .claude-plugin/plugin.json, hooks/hooks.json, and ${CLAUDE_PLUGIN_ROOT} paths"
date: 2026-04-01
agent: Claude Code
---

## Problem

The framework required users to `git clone` the repo and manually copy `.claude/` directories into their projects. This meant:
- No one-command install
- No automatic updates
- Manual `.claude/settings.json` configuration for hooks
- No clean separation between plugin components and project state

## Solution

Converted to a Claude Code plugin (v2.0.0):

1. **Plugin manifest**: `.claude-plugin/plugin.json` with name, version, hooks path
2. **Component layout**: moved `agents/`, `skills/`, `commands/` from `.claude/` to root level
3. **Hook registration**: `hooks/hooks.json` with `${CLAUDE_PLUGIN_ROOT}/hooks/handlers/` paths (replaces `.claude/settings.json`)
4. **Env vars**: root `settings.json` for `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`
5. **Skill injection**: all `$(cat .claude/skills/...)` replaced with `$(cat ${CLAUDE_PLUGIN_ROOT}/skills/...)` in 8 files
6. **Bootstrapping**: session-start.sh creates `ops/` and `.claude/` directories on first run, copies skeleton templates
7. **Templates**: `templates/CLAUDE.md` and `templates/ops/` for new project setup

## Key lessons

- `${CLAUDE_PLUGIN_ROOT}` is substituted by Claude Code in skill/agent/command content AND available as env var in hook scripts
- Project-local state files (`.claude/ship-loop.local.md`, `.claude/context-monitor.local.md`) must stay project-relative — hooks execute in project CWD
- All 3 hook scripts need `mkdir -p .claude` since the directory no longer exists in the repo
- `ops/` is project state, not plugin state — must be bootstrapped per-project, not shipped with plugin
- The ops/ bootstrapping guard should always ensure subdirectories exist (`mkdir -p ops/solutions ops/decisions ops/archive`), not just check if `ops/` exists

## Prevention

- When adding new hooks, always include `mkdir -p` for any directories they write to
- When referencing plugin files in commands/agents, always use `${CLAUDE_PLUGIN_ROOT}/` prefix
- When referencing project files (ops/, .claude/*.local.md), use relative paths (hooks run in project CWD)
