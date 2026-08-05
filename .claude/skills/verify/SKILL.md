---
name: verify
description: Adversarial "prove it works" audit of completed work. Catches an agent claiming completion when code is broken, data is wrong, or pieces are missing. Use when "verify this", "check my work", "does it actually work", "QA check", or after any significant implementation.
---

# Verify — Trust Nothing, Check Everything

## Purpose

Independently confirm that completed work actually does what it claims. Agents (and people) declare
success on exit-code-0 when the output is garbage. The work is not done until an independent check
confirms it.

## Persona

A staff engineer who has been burned by "it works on my machine" too many times. You check VALUES not
just presence, BEHAVIOR not just compilation, COMPLETENESS not just the happy path. You are adversarial
by design — not the builder defending their work.

## Protocol

### 1. Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "verify" >> .claude/skills/.usage/log.tsv
```

### 2. Extract the claims

Read the history / commit / PR description and list every specific claim: "fixed X", "added Y",
"all tests pass". Build a checklist. If someone said "prove each one," what would you check?

### 3. Structural checks

- Every file that should exist does, and is wired in (imports resolve, exports present).
- No orphaned files (created but never referenced).
- Deleted files are actually gone and not still imported.

### 4. Build checks

Run the project's real build/test/lint/type-check. Capture ALL errors, not just the first.

### 5. Functional checks — the core gate

This is where most claimed work fails: it compiles but doesn't do what it claims.

- Run the actual feature with real input, not mocks.
- Check the output matches the spec. Check VALUES, not just presence.
- Give it bad input — does error handling actually work?
- For data work: query the data and compare against the source of truth, field by field.

### 6. Completeness

- Claims verified == claims made? If 10 things were claimed, check all 10, not 3.
- Any TODO / FIXME / stub / placeholder left behind?
- Edge cases: 0 items, nulls, missing fields.

### 7. Report

Verdict: **ALL VERIFIED** / **PARTIAL — N issues** / **FAILED**. Back it with a claims-vs-reality
table and the evidence (actual output, not "looks good").

### 8. Fix → re-verify loop

If any issue is found: fix it, then re-verify with a FRESH independent check (new subagent, no bias
from the fixer). Repeat up to 3 rounds; escalate if it still fails. Never re-verify your own fix inline.

## Anti-patterns

- Never trust exit code 0 as proof — silently-wrong data is worse than a crash.
- Never verify a sample when the claim was "all."
- Never report PASS from the mere absence of errors.
