# execute-one-step — Internal Engine

Read by `rapid-execute`, `resume`, and `rerun-step` before dispatching.
**Not user-invokable.** No YAML frontmatter.

## Inputs
- `step_file`: absolute path to a `task-NN-*.md` in `.rapid/plans/<feature>/`
- `executor`: `claude` | `opencode` (from step file, overridden by verbal toggle)
- `model`: model ID from the step file

## Behavior by Kind

### code node
1. Read `step_file`. Extract: Contract, Implementation notes, Files.
2. Dispatch **code-writer subagent** via executor:
   - Model: `model` from step file
   - Executor: see Executor Routing below
   - Prompt template: `code-writer-prompt.md`
   - Inputs (pass as file paths, not pasted content):
     - Step file: `step_file`
     - Report file: `.rapid/run/<node-name>-report.md`
3. Append to `.rapid/ledger.md`:
   `<node-name>: complete <ISO-8601-timestamp>`
4. Return: status (DONE | BLOCKED | NEEDS_CONTEXT) + one-line summary.

### author-test node
1. Read `step_file`. Extract: Contract, Test instructions. **Do not include implementation file paths in the dispatch — the test-writer must not read any implementation.**
2. Dispatch **test-writer subagent** via executor:
   - Model: `model` from step file
   - Executor: see Executor Routing below
   - Prompt template: `test-writer-prompt.md`
   - Inputs: step file path (contract + test instructions only), report file path
3. Append to `.rapid/ledger.md`:
   `<node-name>: complete <ISO-8601-timestamp>`
4. Return status.

### run-test node (the gate)
1. Read `step_file`. Extract: Contract (contains the test command).
2. Run: `<test command from step file Contract section>`
3. If PASS: append to ledger, return DONE.
4. If FAIL — fixer loop (max 3 attempts):
   a. Dispatch **fixer subagent** with:
      - Step file path (contract = arbiter)
      - Failing test output
      - Paths of implementation files under test
      - For integration/smoke: also include the unit code file paths the integration composes
      - Prompt template: `fixer-prompt.md`
      - Report file: `.rapid/run/<node-name>-fix-<attempt>-report.md`
      - Model: step file's declared model (escalate to Balanced tier from `skills/model-profiles/SKILL.md` on attempt 2+)
   b. Re-run test command.
   c. If PASS: append to ledger, return DONE.
5. After 3 failed attempts: return BLOCKED (do not mark ledger).

### bootstrap node
1. Read `step_file`. Extract: Bootstrap targets, Files.
2. Dispatch **bootstrap subagent** via executor:
   - Model: `model` from step file
   - Executor: see Executor Routing above
   - Prompt template: `bootstrap-prompt.md`
   - Inputs: step file path, report file path
3. Verify scaffolding succeeded:
   a. Read the step file's **Files** section to get the expected file list. For each file: `[ -f "<path>" ] && echo "OK: <path>" || echo "MISSING: <path>"`.
   b. If any file is MISSING: return BLOCKED with list of missing files.
   c. If all present, run: `go vet ./...` to confirm minimal project validity.
   d. If `go vet` fails with non-trivial errors (not "no Go files"): return BLOCKED with vet output.
4. Append to `.rapid/ledger.md`:
   `<node-name>: complete <ISO-8601-timestamp>`
5. Return: status (DONE | BLOCKED | NEEDS_CONTEXT) + one-line summary.

## Executor Routing

Detect environment:
- Check `$OPENCODE_NATIVE` env var (set to `1` by the `using-superpowers` bootstrap in native opencode TUI sessions).
- Check for task-dispatch capability: if the agent has access to a `task` or `subagent` tool for dispatching subagents.
- If `OPENCODE_NATIVE=1` OR Task tool is available → native opencode TUI mode.
  - Always dispatch via Task tool subagent.
  - Prepend prompt preamble: "Model assigned: <model>. Consult skills/model-profiles/SKILL.md for capability context."
  - Fallback: if subagent returns BLOCKED, retry once. If still BLOCKED, escalate to user.
- Otherwise (external harness like Claude Code):
  - If `Executor: opencode` → shell out: `opencode run --model <model> --file <step_file> > <report_file>`
    On non-zero exit or BLOCKED: retry once via native subagent as fallback.
    Log: `echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] opencode BLOCKED for <node>, retrying with Claude" >> .rapid/run/opencode-executor.log`
  - If `Executor: claude` → dispatch native subagent via Task tool (same as opencode TUI default).

## Ledger Format
`.rapid/ledger.md` — one line per completed node, appended:
```
task-01-code-user-store: complete 2026-06-27T14:32:01Z
task-02-author-test-user-store: complete 2026-06-27T14:32:03Z
```
The ledger is the recovery anchor. Read it at the start of `resume` to skip completed nodes.
