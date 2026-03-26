---
title: "grep -oP fails silently on macOS — use POSIX sed instead"
date: 2026-03-26
tags: [hooks, posix, macos, bsd, grep, sed]
agent: Claude Code (Opus)
---

## Problem
ship-loop.sh and context-monitor.sh used `grep -oP 'key: \K\d+'` to extract numeric values from YAML-like state files. On macOS, BSD grep lacks the `-P` (Perl regex) flag entirely. The `\K` lookbehind is a Perl-only feature.

The failure was **silent** because each grep was wrapped with `|| echo "0"` fallback. This meant:
- `ITERATION` was always 0 → ship-loop never tracked iterations
- `consecutive_reads` was always 0 → context-monitor never detected analysis paralysis

## Root cause
GNU grep (Linux) supports `-P` for Perl-compatible regex. BSD grep (macOS) does not. The framework was developed/tested in an environment where this distinction wasn't caught.

## Solution
Replace all `grep -oP 'key: \K\d+'` patterns with POSIX-compatible `sed`:

```bash
# Before (GNU-only):
ITERATION=$(grep -oP 'iteration: \K\d+' "$STATE_FILE" 2>/dev/null || echo "0")

# After (POSIX):
ITERATION=$(sed -n 's/^iteration: \([0-9]*\).*/\1/p' "$STATE_FILE" 2>/dev/null)
ITERATION="${ITERATION:-0}"
```

The `${VAR:-default}` pattern handles empty results more cleanly than `|| echo "0"`.

## Prevention
- Never use `grep -P` or `grep -oP` in hooks or scripts
- Use `sed -n 's/pattern/\1/p'` with basic regular expressions (BRE) for extraction
- Test hooks on both macOS and Linux before committing
