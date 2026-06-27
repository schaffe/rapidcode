---
name: rapid-spec
description: Use when starting any new build — turns an idea into a reviewed spec in under 5 minutes
---

# Rapid Spec

Get to a reviewable, editable spec in under 5 minutes. No ceremony.

**Announce at start:** "I'm using rapid-spec to write the spec."

## Checklist

1. Read project context (1 min max)
2. Ask up to 3 clarifying questions — one at a time, stop when you understand the deliverable
3. Write the spec to `.rapid/specs/YYYY-MM-DD-<topic>.md`
4. Review loop — iterate spec with user until they confirm
5. Invoke `rapidcode:plan-as-folder`

## 1. Project Context

Spend no more than 1 minute. Check:
- Existing file structure (`ls` key dirs)
- Recent `git log --oneline -5`
- Any existing `.rapid/specs/` that might be related

## 2. Clarifying Questions

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a question can be answered by exploring the codebase, explore the codebase instead.

Stop as soon as you understand:
- What the deliverable is (the concrete runnable thing)
- Hard constraints (tech stack, existing integrations, hard limits)
- Success criterion (how will you know it works — a specific command or action)

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

## 4. Review Loop

After writing the spec, say:

> "Spec written to `.rapid/specs/<filename>`. Does this look right? What's missing, wrong, or unclear?"

Then iterate:
- Ask follow-up questions one at a time based on their feedback
- Update the spec file after each round of feedback
- Continue until the user explicitly says they're done (e.g. "looks good", "ship it", "done")

## 5. Transition

When the user says they're done editing, invoke `rapidcode:plan-as-folder`.
Pass the spec file path as context.
