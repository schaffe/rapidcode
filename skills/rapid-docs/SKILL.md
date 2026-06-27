---
name: rapid-docs
description: Use after rapid-execute to update README (and other stale docs) from the spec and executed plan — no user interview, fully autonomous.
---

# Rapid Docs

Translate the latest spec + plan + working-tree diff into updated user-facing docs.

**Announce at start:** "I'm using rapid-docs to update the docs from the spec."

> Runs **inline in the rapid-execute orchestrator** — reuses its context and capable model. Do NOT dispatch as a fresh subagent.

## Checklist

1. Locate sources
2. Determine doc targets
3. Synthesize from spec → docs
4. Update in place
5. Report

## 1. Locate Sources

- **Spec**: most-recently modified file under `.rapid/specs/` matching the current feature context.
- **Plan**: `.rapid/plans/<spec-basename>/` — read `00-overview.md` and the task files.
- **Diff**: `git diff $(git merge-base HEAD main)` — the full working-tree change vs merge-base.

## 2. Determine Doc Targets

- **README.md** always (create if absent; update if present).
- Any existing tracked doc made stale by the diff: CHANGELOG, `docs/` usage files, man pages. Detect staleness by checking whether the doc describes an interface or command that the diff changed.
- **Do NOT create new docs** except README. Update only files that already exist.

## 3. Synthesize: Spec → Docs

Map spec sections to doc content:

| Spec section | Doc output |
|---|---|
| Goal | README purpose / intro paragraph |
| Contracts (interface + examples) | Usage / API section with exact examples |
| Success Criterion | "Run / Verify" command block |

Rules:
- Document only what the diff **confirms was built** — never document planned or future work.
- Preserve exact CLI flags, function signatures, and example output from the spec's Contract.
- Do not paraphrase contracts; copy them verbatim into examples, then wrap with prose.

## 4. Update In Place

- **Preserve existing README structure and voice.** Add or update sections; do not reorder or rewrite sections the diff did not touch.
- Match surrounding heading level, code-fence language tags, and list style.
- If README does not exist, create a minimal one: title → purpose → usage → verify.
- For CHANGELOG: prepend a new entry under the current version heading. Do not reformat existing entries.

## 5. Report

Print one line per file, e.g.:

```
updated  README.md  — added "Export to CSV" usage section
created  README.md  — new file from spec
updated  CHANGELOG  — prepended v1.2.0 entry
```

Then stop. Ask the user nothing.

## Rules

- **Do NOT interview the user.** Fully autonomous — derive everything from spec, plan, and diff.
- **Never commit.** Doc writes are staged by the rapid-execute orchestrator, not this skill.
- **Never hallucinate interfaces.** If the spec's Contract and the diff conflict, use the diff (what was actually built).
- **Scope to the feature.** Do not update docs for unrelated features found elsewhere in the repo.
