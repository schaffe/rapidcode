---
name: lean-spec
description: Use when starting any new build — turns an idea into a reviewed spec in under 5 minutes
---

# Lean Spec

Get to a reviewable, editable spec in under 5 minutes. No ceremony.

**Announce at start:** "I'm using lean-spec to write the spec."

## Checklist

1. Read project context (1 min max)
2. Ask up to 3 clarifying questions — one at a time, stop when you understand the deliverable
3. Write the spec to `.rapid/specs/YYYY-MM-DD-<topic>.md`
4. Manual-edit gate — wait for user confirmation
5. Invoke `rapidcode:plan-as-folder`

## 1. Project Context

Spend no more than 1 minute. Check:
- Existing file structure (`ls` key dirs)
- Recent `git log --oneline -5`
- Any existing `.rapid/specs/` that might be related

## 2. Clarifying Questions

Ask **at most 3 questions**, one at a time. Stop as soon as you understand:
- What the deliverable is (the concrete runnable thing)
- Hard constraints (tech stack, existing integrations, hard limits)
- Success criterion (how will you know it works — a specific command or action)

**No visual companion. No exploring alternative approaches. No proposing options.**
If the user provides enough context up front, ask fewer than 3 questions.

## 3. Write the Spec

Save to `.rapid/specs/YYYY-MM-DD-<topic>.md`. Max 2 pages. Use exactly this format:

```markdown
# <Feature> Spec

**Goal:** One sentence — the runnable thing being built.

**Constraints:**
- [hard constraint: tech, API, version, limit]

## Contracts

### <Unit Name>
**Interface:** `functionName(param: Type): ReturnType`
**Example:** `functionName("input") → "output"`
**Notes:** [edge cases or invariants, if any]

### <Integration: UnitA + UnitB>
**Interface:** [how A and B compose — what calls what]
**Example:** [end-to-end I/O through the integration]
**Test command:** `<exact shell command to run integration tests>`

## Success Criterion
`<exact command or user action>` → `<expected observable output>`
```

Rules:
- Every unit the spec requires gets a Contract section
- Every contract has a concrete input→output example (the test-writer's arbiter)
- No "TBD", no "handle edge cases", no vague requirements
- No commit

## 4. Manual-Edit Gate

After writing the spec, say **exactly**:

> "Spec written to `.rapid/specs/<filename>`. **Edit it now** — fix anything wrong, add anything missing, cut anything not needed. Tell me when you're done."

Wait. Do not proceed until the user explicitly says they're done editing.

## 5. Transition

When the user says they're done editing, invoke `rapidcode:plan-as-folder`.
Pass the spec file path as context.
