# rapidcode

A speed-first development methodology for coding agents. Get a running, tested prototype in under 15 minutes, then harden it to reviewer-clean in the background.

## How it works

The moment you describe what you want to build, rapidcode kicks in. It asks only the questions that change the build (at most three), writes a lean spec, and pauses so you can edit it by hand. Once you confirm, it decomposes the spec into a parallel DAG of self-contained step files and executes them — code and tests authored independently from the same contracts, in parallel waves. Phase 1 ends when every test is green and the prototype runs. You start playing with it immediately while Phase 2 hardens it autonomously via a review loop.

## The Workflow

```
/rapidcode:lean-spec        # ≤3 questions → spec → manual-edit gate
        ↓
/rapidcode:plan-as-folder   # spec → DAG of step files under .rapid/plans/
        ↓                     (manual-edit gate before execution)
/rapidcode:rapid-execute    # parallel wave-scheduler → Phase 1 done → Phase 2 review loop
```

### Phase 1 — Time-critical (≤15 min)

Each step file declares a node kind, a model, and a contract. The wave-scheduler dispatches all independent nodes in parallel:

- **Wave 1 (all in parallel):** every `code` node (implements a unit) + every `author-test` node (writes tests from the contract, never reads the implementation)
- **Subsequent waves:** `run-test` gates fire as their deps land — unit tests, then integration tests, then the smoke test (DAG sink)

When every `run-test` node is green → prototype is running, nothing committed.

### Phase 2 — Background (not time-critical)

Whole-branch review (working-tree diff vs merge base) → fix → re-review until zero Critical/Important findings → notify. Runs autonomously while you play with the prototype.

## Skills

### Core workflow
| Skill | When to use |
|---|---|
| `lean-spec` | Starting any new build |
| `plan-as-folder` | After lean-spec — auto-invoked |
| `rapid-execute` | After plan-as-folder confirms — auto-invoked |

### Recovery & iteration
| Skill | When to use |
|---|---|
| `resume` | After a rapid-execute was aborted or interrupted |
| `rerun-step` | Re-run one specific step file in isolation |
| `opencode-executor` | Toggle free-model execution via opencode CLI for the session |

### Engineering disciplines
| Skill | When to use |
|---|---|
| `test-driven-development` | Implementing any feature or bugfix |
| `systematic-debugging` | Any bug, test failure, or unexpected behavior |
| `verification-before-completion` | Before claiming work is complete |
| `requesting-code-review` | Completing features, before merging |
| `receiving-code-review` | Responding to review feedback |
| `dispatching-parallel-agents` | 2+ independent tasks without shared state |
| `using-git-worktrees` | Feature work that needs isolation |
| `finishing-a-development-branch` | Implementation complete, time to integrate |
| `writing-skills` | Creating or editing skills |

## Installation (Claude Code)

This is a local plugin. From this repo:

```bash
/plugin install rapidcode
/reload-plugins
```

Or install directly from the directory:

```bash
/plugin install /path/to/rapidcode
```

## Artifact layout

All runtime artifacts are gitignored under `.rapid/`:

```
.rapid/
  specs/<date>-<topic>.md       # lean-spec output — edit before planning
  plans/<feature>/
    00-overview.md              # DAG table, goal, constraints
    task-NN-<kind>-<name>.md    # self-contained step files
  ledger.md                     # per-step completion record (resume anchor)
  run/                          # subagent reports, diffs, review packages
```

Nothing is committed by the workflow. You commit when satisfied.

## Step file format

Each step file declares its node kind, dependencies, model, and contract:

```markdown
# Task NN: <name>

**Kind:** code | author-test | run-test
**Test-scope:** unit | integration | smoke   (omit for code nodes)
**Depends-on:** []                           ([] = Wave-1 root)
**Executor:** claude
**Model:** claude-haiku-4-5-20251001

**Files:**
- Create: `src/exact/path.ts`

## Contract
`functionName(input: Type): ReturnType`
Example: `functionName("x") → "y"`

## Implementation notes   (code nodes only)
## Test instructions      (author-test nodes only)
```

- `code` and `author-test` nodes have empty `Depends-on` — they run in Wave 1 in parallel
- `run-test` nodes depend on their `code` + `author-test` nodes
- Integration and smoke `run-test` nodes are the DAG's later waves and sink

## Design principles

- **Rigor lives in the plan.** Contracts + independent test authorship replace per-task reviewer subagents.
- **Tests are the gate.** Code and tests are written from the same contract by separate subagents. Tests verify the contract, not the implementation's bugs.
- **No eager commits.** Phase 1 builds in the working tree. You decide when to commit.
- **Cheap models, explicitly.** Every step file declares its model. Default: haiku. Escalate only where the plan says so.
- **Minimal reports.** Subagents write detail to `.rapid/run/` and return status only.

## Recovery

If `rapid-execute` is interrupted:

```bash
/rapidcode:resume .rapid/plans/<feature>/
```

Reads `.rapid/ledger.md`, skips completed nodes, resumes from the first incomplete wave.

To re-run a single node:

```bash
/rapidcode:rerun-step .rapid/plans/<feature>/task-NN-<name>.md
```

## Free models via opencode

Say "use free models" or "use opencode" before running `rapid-execute` — or invoke the skill directly:

```bash
/rapidcode:opencode-executor
```

This routes all step dispatch through the opencode CLI using the model declared in each step file. Claude is the automatic fallback on failure.

## License

MIT License — see LICENSE file for details.
