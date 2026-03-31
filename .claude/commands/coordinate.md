---
description: "Run the full Phase 0-6 multi-agent sprint cycle for a goal."
argument-hint: "<goal description>"
---

You are running a full multi-agent sprint cycle. Follow docs/multi-agent-framework.md through ALL phases.

## Goal
$ARGUMENTS

## Activation

Write the ship-loop state file to activate the inner loop guard:

```
Create .claude/ship-loop.local.md with:
---
iteration: 0
max_iterations: 5
---
[The goal and context]
```

## Execute the full lifecycle

### Phase 0: Codebase analysis
Invoke Gemini with codebase-mapping skill (skip if unnecessary):
```bash
gemini -p "$(cat .claude/skills/codebase-mapping/SKILL.md) Analyze the full codebase. Write to ops/ARCHITECTURE.md, ops/MEMORY.md (append), ops/CONTRACTS.md (append)." > /tmp/gemini_phase0.txt 2>&1 &
```
Read updated ops/ files after completion.

### Pre-Plan: Institutional knowledge
Spawn `learnings-researcher` agent to search ops/solutions/ and ops/decisions/.

### Phase 1: Planning
Follow `writing-plans` and `shadow-path-tracing` skills. Embed CONTRACTS.md types. Group into waves. Write TASKS.md.

### Phase 1.5: Plan validation
Spawn `plan-checker` agent. Iterate until APPROVED (max 3 rounds).

### Phase 2: Build
Use wave orchestration. Subagent mode for < 5 tasks, agent team mode for 5+.
Run `integration-verifier` between waves. Apply risk scoring.

### Phase 3: Parallel review
Launch Gemini + Codex (background bash) + Claude review agents simultaneously:
- security-sentinel agent
- performance-oracle agent
- code-simplicity-reviewer agent

### Phase 4: Process reviews
Spawn `findings-synthesizer`. Apply `iterative-refinement` skill. Fix P1+P2. Loop if needed (max 3 cycles).

### Phase 5: Test
Spawn `test-gap-analyzer`. Invoke Codex with TDD skill. Fix failures until green.

### Phase 6: Wrap up
- Apply `knowledge-compounding` (document to ops/solutions/ if non-trivial)
- Update ops/CHANGELOG.md, ops/MEMORY.md, ops/TASKS.md
- Archive review files to ops/archive/[date]/
- Apply `verification-before-completion` checklist
- Write ops/STATE.md
- Remove .claude/ship-loop.local.md
- Sprint summary for user

Only when ALL work is verified complete:
<promise>DONE</promise>
