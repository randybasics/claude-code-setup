---
name: plan-review
description: Unbiased senior-engineer review of an implementation plan or design doc BEFORE any code is written. Use when "review this plan", "engineer review", "get feedback on this design", or before starting a non-trivial build.
---

# Plan Review — Independent Senior Engineer

## Purpose

Stress-test a plan *before* code exists, when changing course is still cheap. The reviewer must be
independent of the author — an agent reviewing its own plan is biased and defeats the point. Run this
as a separate subagent, then present the verdict verbatim.

## When to Activate

- "review this plan", "engineer review", "feedback on this design"
- Before any implementation that touches more than one file or has non-obvious trade-offs

## Protocol

### 1. Restate the plan in one paragraph

If you can't restate it crisply, the plan is too vague to review — say so and ask for specifics.

### 2. Review across five lenses

| Lens | Questions |
|---|---|
| **Correctness** | Does the approach actually solve the stated problem? What breaks it? |
| **Simplicity** | Is there a materially simpler approach that does the same job? |
| **Blast radius** | What else does this touch? What could it break that isn't obvious? |
| **Edge cases** | Empty input, nulls, concurrency, failure mid-operation, rollback path |
| **Altitude** | Is the fix at the right depth, or a bandaid layered on shared infra? |

### 2.5 Database change gate (conditional — Principal DBA + Sr. Data Engineer)

If the plan touches the data layer — a table/column/type, a migration, RLS/grants/roles, an
index/view/trigger/function, a new query or ORM model, or a backfill — a dedicated Principal DBA +
Sr. Data Engineer review is **required** in addition to the five lenses. Read
[`references/dba-review.md`](references/dba-review.md) and produce its `## Database Architecture
Review` section, covering BOTH the overall database architecture the change lands on AND the
specific changes, with its own DB verdict (APPROVE / APPROVE WITH FIXES / BLOCK). Run it as its own
independent subagent when orchestrated; inline as a distinct section under a hard persona switch
otherwise. No DB changes → record "No DB changes — DBA review skipped" and continue. A BLOCK caps
the overall verdict at SHIP WITH FIXES or lower.

### 3. Give a verdict

One of:
- **SHIP IT** — sound, proceed
- **SHIP WITH FIXES** — proceed after applying the listed fixes
- **NEEDS REWORK** — the approach has a structural problem; rethink before building
- **STOP** — the premise is wrong; escalate to the human

### 4. Be specific and actionable

Every concern names the exact risk and a concrete alternative. No theoretical critique — if you can't
name what would go wrong and what to do instead, drop it.

## Rules

- The plan's author must NOT run this on their own plan — use an independent reviewer.
- Present the review unfiltered. Don't soften it.
- Balance speed and rigor: actionable feedback, not an academic audit.
