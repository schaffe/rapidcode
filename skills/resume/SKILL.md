---
name: resume
description: Use after a rapid-execute was aborted or interrupted — resumes from the last completed node
---

# Resume

Resume a `rapid-execute` run that was aborted, interrupted, or hit a BLOCKED node.

**Announce at start:** "I'm using resume to continue the interrupted run."

## Arguments

`/rapidcode:resume <plan-dir>` — path to the plan folder, e.g. `.rapid/plans/my-feature/`

## Process

1. **Read `.rapid/ledger.md`** — list every node marked `complete`.
   If no ledger exists: resume is equivalent to a fresh `rapidcode:rapid-execute <plan-dir>` — proceed with all nodes.

2. **Read every step file** in `<plan-dir>`. Find nodes NOT in the ledger.

3. **Rebuild the remaining DAG subgraph:**
   - Start from incomplete nodes whose all deps are already in the ledger (ready to run immediately).
   - Include their transitive dependents (anything downstream that isn't complete yet).

4. **Re-enter the wave-scheduler** from rapid-execute §3, feeding only the remaining subgraph.
   The already-complete nodes count as "satisfied" for dependency purposes.

5. **Continue with Phase 1 → Phase 2** exactly as `rapid-execute` would.

## Recovery guarantees
- Re-running a completed node is safe (execute-one-step is idempotent for its file outputs).
- The ledger is the anchor. If the ledger was destroyed (`git clean -fdx`), treat it as absent — re-run everything.
- Nothing is committed by resume.

## Rules
- Never re-run a node that appears in the ledger as `complete` (trust the ledger).
- Read all step files from disk (not from context).
