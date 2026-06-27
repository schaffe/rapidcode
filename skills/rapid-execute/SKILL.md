---
name: rapid-execute
description: Use to execute a plan-as-folder DAG — runs the prototype to green tests (Phase 1), then review-cleans it autonomously (Phase 2)
---

# Rapid Execute

Execute a `.rapid/plans/<spec-basename>/` step-file DAG to a tested prototype, then review-clean it.

**Announce at start:** "I'm using rapid-execute to execute the plan."

## Arguments

`/rapidcode:rapid-execute <plan-dir>` — path to the plan folder, e.g. `.rapid/plans/2026-06-27-my-feature/`

Optional verbal flags (speak them before invoking):
- "use free models" / "use opencode" → invoke `rapidcode:opencode-executor` first, then run this skill

## Checklist

1. Reload spec and plan from disk
2. Build DAG + check ledger
3. Phase 1: wave-schedule all nodes to green
4. Phase 2: review loop until reviewer-clean, invoke rapid-docs, then notify

## 1. Reload from Disk

**Always read from disk. Never trust in-context copies.**

1. Read `<plan-dir>/00-overview.md` — note the goal and node table.
2. Read every `task-NN-*.md` in `<plan-dir>` — build the full node list with their Kind, Depends-on, Executor, and Model.
3. Read `.rapid/ledger.md` if it exists — any node listed as `complete` is already done. Skip it in execution.

Read `skills/rapid-execute/execute-one-step.md` to confirm the engine contract before dispatching.

## 2. Build DAG

From every node's `Depends-on:` list:
- Wave 1 = nodes with empty `Depends-on` (all `code` and `author-test` nodes)
- Wave N+1 = nodes whose all deps appear in Waves 1..N

**Verify:** no dep cycles. If any node depends on itself (directly or transitively), stop and report.
**Skip** any node already marked complete in the ledger.

## 3. Phase 1: Wave-Schedule

Initialize `.rapid/run/` directory: `mkdir -p .rapid/run`

For each wave (in order, skipping fully-complete waves):
1. **Dispatch ALL non-skipped nodes in the wave in parallel** via `execute-one-step` (read `execute-one-step.md`).
   - Pass each node: step file path, executor (from step file or opencode toggle), model (from step file).
2. Wait for all dispatched nodes to return status.
3. Handle statuses:
   - **DONE**: continue.
   - **NEEDS_CONTEXT**: provide the missing context, re-dispatch that node (same wave, same model).
   - **BLOCKED**: stop Phase 1. Report to user: which node, what the blocker is. Wait for resolution.
4. When all nodes in the wave are DONE: release the next wave.

**Do not pause between waves to check in.** Execute until all waves are complete or a BLOCKED occurs.

**Phase 1 complete when:** every `run-test` node (unit + integration + smoke) is DONE.

Announce:
> "Phase 1 complete — all tests green, prototype is running. Nothing committed. Start playing with it. Beginning Phase 2 review now."

## 4. Phase 2: Review Loop

Phase 2 runs autonomously. The user does not need to be present.

**Reviewer model: `claude-opus-4-8` (most capable — Phase 2 is not time-critical).**
**Fixer model: `claude-sonnet-4-6`.**

```
loop:
  1. Generate diff:
     Run: skills/rapid-execute/scripts/review-package
     This writes the diff to .rapid/run/review-<timestamp>.diff and prints the path.

  2. Dispatch code-reviewer subagent (model: claude-opus-4-8):
     Prompt: "You are doing a whole-branch code review. Your goal: correctness and integration test coverage.

     Diff file: <printed path from step 1>

     Return a prioritized findings list:
     - Critical: correctness bugs, broken contracts, missing integration tests for cross-unit behavior
     - Important: weak tests (assert nothing, mirror the impl), unclear interfaces, real non-blocking issues
     - Minor: style, nitpicks (list but do not fix)

     Focus: do the integration tests actually verify cross-unit behavior? Are all contract examples exercised?
     Do not fix anything. Return findings only."

  3. If reviewer returns zero Critical + zero Important findings:
     break

  4. Dispatch fixer subagent (model: claude-sonnet-4-6) with ALL Critical + Important findings at once:
     - Never dispatch one fixer per finding — one fixer gets the complete list.
     - Fixer re-runs the full test suite after fixing.
     - Fixer appends results to .rapid/run/phase2-fix-<N>-report.md.

  5. Continue loop.
```

On loop exit, stage the work (do not commit):
> Run: `git add -A`

Staging makes the full change set easy to review and commit, without committing it for the user.

Then invoke rapid-docs:
> Invoke: `rapidcode:rapid-docs`

Then re-stage any doc changes:
> Run: `git add -A`

Then notify:
> "Phase 2 complete — zero Critical/Important findings. The prototype is review-clean, docs updated. Changes are staged but not committed. Commit whenever you're satisfied."

## Rules
- **Never commit** in Phase 1 or Phase 2. Staging (`git add -A`) and invoking `rapidcode:rapid-docs` at the end of Phase 2 are fine; committing is the user's call.
- **Never pause between nodes** to check in — execute until done or BLOCKED.
- **Always read files from disk** — never paste step file contents into dispatch prompts.
- **opencode toggle**: if the user invoked `rapidcode:opencode-executor` this session, all dispatch routes through opencode. Claude is the fallback on BLOCKED.
