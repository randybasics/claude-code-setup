---
name: simplify
description: Reuse / simplification / efficiency / altitude cleanup pass on a diff. Quality only — it does not hunt for correctness bugs (use a code review for that). Use when "simplify this", "clean up the diff", "quality pass".
---

# Simplify — Quality Cleanup Pass

## Purpose

Improve the quality of changed code — reuse, simplification, efficiency, altitude — then apply the
fixes. This is NOT a bug hunt; correctness review is a separate job.

## Phase 0 — Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "simplify" >> .claude/skills/.usage/log.tsv
```

## Phase 1 — Gather the diff

Get the unified diff under review:

```bash
git diff @{upstream}...HEAD   # or: git diff main...HEAD, or: git diff HEAD~1
git diff HEAD                  # also include uncommitted work if any
```

If a PR number, branch, or path was passed, review that instead.

## Phase 2 — Review across four lenses

**Reuse** — Does the new code re-implement something the codebase already has? Grep shared/utility
modules and adjacent files; name the existing helper to call instead.

**Simplification** — Redundant or derivable state, copy-paste with slight variation, deep nesting,
dead code left behind. Name the simpler form that does the same job.

**Efficiency** — Redundant computation or repeated I/O, independent operations run sequentially,
blocking work added to a hot path. Name the cheaper alternative.

**Altitude** — Is each change at the right depth, or a fragile bandaid? Special cases layered on
shared infrastructure usually mean the fix isn't deep enough — prefer generalizing the mechanism.

## Phase 3 — Apply the fixes

Dedup findings that point at the same line, then fix each one. Skip any finding whose fix would
change intended behavior, reach well outside the diff, or that you judge a false positive — note the
skip rather than arguing with it. Finish with a short summary of what was fixed and what was skipped.

## Rule

Quality only. If you spot a real correctness bug, flag it separately — don't fix it under the banner
of "simplification."
