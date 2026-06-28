---
name: plan-as-folder
description: Use after rapid-spec to decompose a spec into a DAG of step files under .rapid/plans/
---

# Plan as Folder

Decompose the approved spec into a folder of self-contained step files — the inputs that `rapid-execute` wave-schedules in parallel.

**Announce at start:** "I'm using plan-as-folder to generate the step-file DAG."

## Checklist

1. Read the spec from disk
2. List every unit and its contract
3. Decompose into code + author-test + run-test nodes
4. Wire DAG edges
5. Assign models
6. Write step files + 00-overview.md
7. Confirm and execute

## 1. Read Spec

Always read from `.rapid/specs/` — the user may have edited it. Never rely on an in-context copy.

## 2. List Units

From the spec's Contracts section:
- Every named unit → one set of code + author-test + run-test nodes
- Every named integration scope → one author-test + one run-test node (no code node — integration tests compose existing units)
- One smoke test (from the Success Criterion) → one author-test + one run-test node

## 3. Decompose into Nodes

**Per unit:**
- `task-NN-code-<unit>.md` — implements the unit
- `task-NN-author-test-<unit>.md` — writes unit tests from the contract
- `task-NN-run-test-<unit>.md` — runs the unit tests; deps = [code node, author-test node]

**Per integration scope:**
- `task-NN-author-test-<scope>.md` — writes integration tests from the contract; deps = []
- `task-NN-run-test-<scope>.md` — runs integration tests; deps = [author-test, all code nodes it composes]

**Smoke test:**
- `task-NN-author-test-smoke.md` — writes the smoke test from the Success Criterion; deps = []
- `task-NN-run-test-smoke.md` — DAG sink; deps = [smoke author-test, all integration run-test nodes]

**Per project (if Go detected — `go.mod` exists or spec declares Go):**
- `task-00-bootstrap-go.md` — scaffolds Makefile, .golangci.yml, go.mod; Depends-on: []

## 4. Wire DAG Edges

Rules:
- `bootstrap` nodes: `Depends-on: []` (Wave-0 root — runs before Wave 1 if present)
- `code` nodes: `Depends-on: []` (Wave-1 root)
- `author-test` nodes: `Depends-on: []` (Wave-1 root — consumes only the contract)
- unit `run-test`: `Depends-on: [code-node, author-test-node]`
- integration `run-test`: `Depends-on: [integration-author-test, code-node-A, code-node-B, ...]`
- smoke `run-test`: `Depends-on: [smoke-author-test, all integration-run-test nodes]`

**Disjoint-file rule:** if two parallel nodes would write the same file, add a `Depends-on` edge to serialize them.

## 5. Assign Models

See `skills/model-profiles/SKILL.md` for capability guidance when choosing `Model:` values.
Default every node to Fast/Cheap tier model.
Escalate to Balanced for standard code and author-test nodes.
Escalate to Deep Reasoning for complex code nodes (>2 files, cross-module, integration concerns).

**Every node MUST declare a model — no omissions.**

## 6. Write Step Files

Create `.rapid/plans/<spec-basename>/` where `<spec-basename>` is the spec filename minus `.md` (format: `YYYY-MM-DD-<topic>`).
The plan folder basename must be identical to the spec filename (minus `.md`). Name files `task-NN-<kind>-<name>.md` (zero-padded NN).

Use this template for every step file:

```markdown
# Task NN: <descriptive name>

**Kind:** code | author-test | run-test | bootstrap
**Test-scope:** unit | integration | smoke   (omit for code nodes)
**Depends-on:** [task-NN, task-NN]           ([] for Wave-1 roots)
**Executor:** claude
**Model:** Fast/Cheap tier model (resolve per skills/model-profiles/SKILL.md for your environment)

**Files:**
- Create: `exact/path/file.ts`
- Modify: `exact/path/existing.ts`

## Contract

`functionName(input: InputType): OutputType`

Example: `functionName({ id: 1 }) → { name: "Alice" }`

(One example per distinct behavior. These are the test-writer's arbiter.)

## Implementation notes   (code nodes only — omit for author-test and run-test)

[Prose guidance on what to implement. No code dumps.]

## Test instructions   (author-test nodes only — omit for code and run-test)

Cover:
- [behavior 1 from Contract]
- [behavior 2 from Contract]

Test command: `<exact shell command to run just these tests>`
```

For `run-test` nodes, the Contract section must include:
```
Test command: `<exact shell command>`
```

For `bootstrap` nodes, omit the Contract section and use **Bootstrap targets** instead:
```
## Bootstrap targets

- Makefile: build, test, lint, fmt, clean, coverage, all
- .golangci.yml: [linter configuration notes]
- go.mod: module name and init instructions
```

### Write 00-overview.md

```markdown
# <Feature> — Plan Overview

**Spec:** `.rapid/specs/<filename>`
**Goal:** [from spec]

## DAG

Wave 1 (parallel): task-01, task-02, task-05, ...
Wave 2:            task-03 (after 01+02), task-07 (after 05+06), ...
Wave N (sink):     task-NN-run-test-smoke

## Node Table

| Node file | Kind | Scope | Model | Depends-on |
|-----------|------|-------|-------|------------|
| task-01-code-... | code | — | haiku | [] |
| task-02-author-test-... | author-test | unit | haiku | [] |
| task-03-run-test-... | run-test | unit | haiku | [01, 02] |
```

## 7. Confirm and Execute

After writing all files, ask:

> "Plan written to `.rapid/plans/<spec-basename>/` — NN step files + 00-overview.md.
> Review the step files now (especially contracts and models) and edit any file directly.
> Ready to execute, or do you want more time to edit?"

If user says ready: invoke `rapidcode:rapid-execute .rapid/plans/<spec-basename>/`
If user says not yet: wait. Re-ask when they return.

No commit.

## Integration Tests

Test plan-as-folder DAG generation behavior:

### DAG Structure
- Go project (with `go.mod` or spec declaring Go): MUST produce `task-00-bootstrap-go.md` in Wave 0
- Non-Go project: MUST NOT produce a bootstrap node
- Wave 1: all `code` and `author-test` nodes have `Depends-on: []`
- Wave N (sink): smoke `run-test` depends on all upstream `run-test` nodes
- No cycles: every node's transitive deps never includes itself

### Step File Format
- Every node declares **Kind**, **Model**, **Executor**, **Depends-on**
- `code` nodes include Contract + Implementation notes + Files
- `author-test` nodes include Contract + Test instructions + Test command, OMIT Implementation notes
- `run-test` nodes include Test command in Contract, OMIT Implementation notes and Test instructions
- `bootstrap` nodes include Bootstrap targets + Files, OMIT Contract

### Model Assignment
- Default: Fast/Cheap tier
- Standard code and author-test: Balanced tier
- Complex code (>2 files, cross-module): Deep Reasoning tier
- Every node has a Model: value — no omissions

### Wave-0 Wiring
- If a bootstrap node exists, it is Wave 0 (runs before Wave 1)
- All Wave-1 nodes have no dependency on the bootstrap node
- Bootstrap node has `Depends-on: []`
