---
name: 5-whys-fix
description: Root-cause analysis for a bug or incident — 5 whys to the real cause, propose 3 solutions, pick one, fix, verify. Use when "why did this break", "root cause", "5 whys", "fix this properly not just the symptom".
---

# 5-Whys Fix — Root Cause, Not Symptom

## Purpose

Stop fixing symptoms. Drive to the actual cause, weigh real options, then fix and prove it. A patch
that hides the symptom while the cause survives just defers the same bug.

## When to Activate

- "why did this break", "root cause this", "5 whys", "fix it properly"
- Any recurring bug, or one whose obvious fix feels like a bandaid

## Protocol

### 1. Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "5-whys-fix" >> .claude/skills/.usage/log.tsv
```

### 2. State the symptom precisely

What is observably wrong? Include the exact error, the input that triggers it, and what *should* have
happened instead. Reproduce it before theorizing.

### 3. Ask "why" five times

Each answer must be evidence-backed (a log line, a code path, a value), not a guess. Stop early only
when you hit a cause that, if fixed, makes the whole chain impossible — not just this instance.

```
Symptom → why? → why? → why? → why? → why? → ROOT CAUSE
```

### 4. Propose three solutions

- **Tactical** — smallest fix at the root cause.
- **Structural** — removes the class of bug, not just this one.
- **Preventive** — a guard/test/type that stops it recurring silently.

Name the trade-off of each (effort vs. durability).

### 5. Pick one and justify

Default to the structural fix unless effort is disproportionate. Say why you chose it.

### 6. Fix and verify

Apply the fix, then prove it independently (see the `verify` skill): the original repro no longer
triggers, AND nothing adjacent broke. Add the preventive guard so it can't silently return.

## Rules

- Every "why" needs evidence — no speculative chains.
- A fix that doesn't change the answer to the deepest "why" is a bandaid; say so.
- Always add the regression guard, or the same incident comes back.
