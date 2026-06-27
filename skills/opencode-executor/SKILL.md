---
name: opencode-executor
description: Toggle free-model execution via the opencode CLI for the current session — use before rapid-execute, resume, or rerun-step
---

# Opencode Executor

Route step dispatch through the opencode CLI using the free/cheap model declared in each step file, instead of Claude subagents.

**This is a session toggle.** Invoking it tells `rapid-execute`, `resume`, and `rerun-step` to route all dispatch through opencode for this session. Claude remains the automatic fallback on failure.

**Announce at start:** "opencode-executor active — routing dispatch through opencode."

## When to Use

Say one of:
- "use free models"
- "use opencode"
- `/rapidcode:opencode-executor`

…then immediately invoke `rapid-execute`, `resume`, or `rerun-step` as normal.

## How Dispatch Works

When opencode-executor is active, `execute-one-step` shells out instead of using the Agent/Task tool:

```bash
opencode run \
  --model <model-from-step-file> \
  --file <step-file-path> \
  > <report-file-path>
```

The step file is the prompt. The model declared in the step file (`Model:` field) is passed directly to opencode.

## Fallback

If opencode returns a non-zero exit code, or if the report file contains `BLOCKED` or `NEEDS_CONTEXT`:

1. Log the event:
   ```bash
   echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] opencode BLOCKED for <node-name>, retrying with Claude" >> .rapid/run/opencode-executor.log
   ```
2. Re-dispatch the same step via the Agent/Task tool using the step file's declared model.
3. If still BLOCKED after the Claude retry: escalate to the user.

## Status Confirmation

After invoking this skill, confirm with:

> "opencode-executor active. All dispatch will route through opencode using the model declared in each step file. Claude is the automatic fallback on failure. Proceed with `/rapidcode:rapid-execute`, `resume`, or `rerun-step`."

## Note on CLI Flags

The exact flags depend on the installed version of opencode. If `opencode run --help` shows different flags than the above, adapt accordingly and note the deviation:

```bash
echo "[$(date -u)] Using adapted flags: <flags>" >> .rapid/run/opencode-executor.log
```

If opencode is not installed or not in `$PATH`, report:
> "opencode CLI not found. Install it first or proceed with the default Claude executor."
