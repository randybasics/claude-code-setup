---
name: implement
description: Full-chain senior-engineer implementation workflow — plan-review → implement → simplify → verify → ship. Use when "implement this", "build this feature", "code this up", "make this work", "full chain".
---

# Implement — Full-Chain Workflow

## Purpose

A senior engineer that never cuts corners. Chains five phases end-to-end; each gates the next. If a
phase finds issues, fix them before proceeding.

## Phase 0 — Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "implement" >> .claude/skills/.usage/log.tsv
```

## Phase 1 — Classify

State the task type up front; it controls every later phase.

| Type | Examples | Branch? | Validate | Ship |
|---|---|---|---|---|
| **Code** | feature, bug fix, refactor | feature branch | test + lint + build | PR |
| **Structure** | move/rename/reorganize files | direct | grep for stale links | commit |
| **Docs** | content, nav, references | direct | links resolve, dates current | commit |
| **Data** | config, JSON, fixtures | direct | read back key fields | commit |

## Phase 2 — Plan & review

1. Write a brief plan: what "done" looks like, files to change, risks, what could break.
2. **Independent plan review** — hand the plan to a separate reviewer (see the `plan-review` skill).
   The builder must not review its own plan. Incorporate the verdict before writing code.

## Phase 3 — Implement

- Match the existing code style. Keep it simple. Handle the edge cases the plan named.
- For code features: write tests alongside.
- For structure: use `git mv`, touch only affected nav lines.

Then validate per the Phase 1 table. All checks must pass before moving on.

## Phase 4 — Simplify (always runs)

Run the `simplify` pass over the full diff — reuse, quality, efficiency — and apply the fixes.
Re-validate after.

## Phase 5 — Verify (independent)

Hand the completed work to an independent verifier (see the `verify` skill). The builder must not
verify its own work. Resolve PARTIAL/FAILED before shipping.

## Phase 6 — Ship

- Security check: grep the staged diff for secrets; abort if any match.
- Commit (feature branch for code, direct for structure/docs/data).
- For code: open a PR and address review feedback until clean.

## Rules

- Classify first — the type controls every phase.
- Simplify and Verify ALWAYS run.
- Plan-review and Verify run as INDEPENDENT steps, never self-review.
- Each phase gates the next; don't skip ahead on failures.
- Small fixes (< 20 lines) may skip Phase 2, but still run Simplify + Verify.
