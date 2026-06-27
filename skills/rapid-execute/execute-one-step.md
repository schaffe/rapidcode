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
      - Model: step file's declared model (escalate to `claude-sonnet-4-6` on attempt 2+)
   b. Re-run test command.
   c. If PASS: append to ledger, return DONE.
5. After 3 failed attempts: return BLOCKED (do not mark ledger).

## Executor Routing
- `Executor: claude` → dispatch via Agent/Task tool.
- `Executor: opencode` OR `rapidcode:opencode-executor` invoked this session →
  shell out: `opencode run --model <model> --file <step_file> > <report_file>`
  On non-zero exit or `BLOCKED` in output: retry once via Claude as fallback.
  Log: `echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] opencode BLOCKED for <node>, retrying with Claude" >> .rapid/run/opencode-executor.log`

## Ledger Format
`.rapid/ledger.md` — one line per completed node, appended:
```
task-01-code-user-store: complete 2026-06-27T14:32:01Z
task-02-author-test-user-store: complete 2026-06-27T14:32:03Z
```
The ledger is the recovery anchor. Read it at the start of `resume` to skip completed nodes.
