---
name: rerun-step
description: Use to re-run one specific step from a plan in isolation, without affecting sibling steps
---

# Rerun Step

Re-run one step file using the same engine as `rapid-execute`. Use when you've manually fixed a unit and need to re-test it, or when a single code or test node failed and you want to retry it alone.

**Announce at start:** "I'm using rerun-step on `<step-file>`."

## Arguments

`/rapidcode:rerun-step <step-file>` — path to a single `task-NN-*.md` file.

Optional verbal flag (same as rapid-execute):
- "use free models" / "use opencode" → routes this dispatch through opencode-executor.

## Process

1. Read `<step-file>` from disk.
2. Read `skills/rapid-execute/execute-one-step.md` (the engine contract).
3. Run `execute-one-step` on the step file with the declared executor + model.
4. On DONE:
   - Update `.rapid/ledger.md`: overwrite any existing entry for this node, or append if absent.
   - Report: "Node `<name>` complete."
5. On BLOCKED: report to the user. Do not update the ledger.

## When to Use
- A `code` node produced a buggy implementation after you've manually patched it — re-run the `run-test` node for that unit.
- A `test-writer` subagent wrote incomplete tests — re-run the `author-test` node.
- You manually edited a unit's code and want to re-run its `run-test` gate.
- A single node was BLOCKED and you've resolved the blocker.

## Rules
- Does **not** check DAG deps — you are responsible for ensuring deps are satisfied.
- Does **not** trigger Phase 2 — use `rapid-execute` or invoke the Phase 2 review manually.
- Safe to run multiple times — the ledger entry is idempotently updated.
- Never commits.
