---
name: auto-skill
description: Routes a plain-language request to the best-matching skill by reading the live .agents/skills/*/SKILL.md catalog, so you never have to remember skill names. Use when the user invokes this skill by name with a request, e.g. "auto-skill: build this feature" or "auto-skill: check my work" (exact invocation syntax is host-specific — /auto-skill in Claude Code, $auto-skill in Codex, by name in Antigravity).
disable-model-invocation: true
---

# Auto-Skill — Plain-Language Skill Router

## Purpose

Say what you want in plain words; this skill reads the current skill catalog, picks the closest match,
states why in one line, then loads and follows it. Stays correct as skills are added or removed — it
never relies on a memorized list. Host-neutral by design: it never assumes a specific slash/dollar
invocation syntax for the skill it routes to, since that syntax differs by host (Claude Code, Codex,
Antigravity).

The request is whatever text the user typed after invoking this skill. If empty, ask "What do you want
to do?" with 2-3 examples, and stop.

## Protocol

### 1. Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "auto-skill" >> .claude/skills/.usage/log.tsv
```

### 2. Load the live skill catalog

Never use a hardcoded list — read what exists right now. `.agents/skills` is the neutral discovery
path every host reads (in this repo it's a symlink to `.claude/skills`, so this works unchanged on
Claude Code too):

```bash
for f in .agents/skills/*/SKILL.md; do
  s=$(basename "$(dirname "$f")")
  d=$(grep -m1 '^description:' "$f" | sed 's/^description:[ >]*//')
  printf '%s :: %s\n' "$s" "$d"
done
```

### 3. Match intent to the single best skill

Compare the request against every description from Step 2. Reason about intent, not keyword overlap.
Starting points for this repo — always re-check against the live catalog, since this table goes stale
as skills are added or removed:

| Request is about... | Routes to |
|---|---|
| build / implement / "make a ___" / code a feature, full chain | `implement` |
| review a plan or design before writing code | `plan-review` |
| clean up a diff for reuse/efficiency (not bug-hunting) | `simplify` |
| prove finished work actually works | `verify` |
| root-cause a recurring bug, "why does this keep breaking" | `5-whys-fix` |
| save context / update project docs | `update-memory-bank` |
| scaffold a new skill or slash command | `skill-factory` |

### 4. Apply tie-breakers

| Situation | Action |
|---|---|
| Two skills are close | Pick the more specific one; name the runner-up in Step 5 |
| Nothing matches well | Say so, list the 2-3 closest, ask which to run (or offer to do the task directly) — never force a bad match |

### 5. Announce, then load and follow the target

Output one line, then read the matched skill's `SKILL.md` in full and follow its protocol from here
on, passing the original request as its input. Don't re-invoke it through a slash/dollar command —
just switch to acting as that skill directly; this is what keeps auto-skill working the same way
regardless of host.

```
→ Matched "<request>" to <skill> — <one-line why>. Loading it now.
  (Also considered: <x>)
```

The target skill's own confirmation gates still apply — auto-skill routes, it does not bypass them.

### 6. Chain only when the request obviously spans phases

Default to one skill. Only name and run a sequence when the request explicitly asks for multiple
phases (e.g. "build X and update the memory bank").

## Rules

- Always re-read the live catalog (Step 2) — a memorized list goes stale.
- One skill by default; chain only when the request explicitly spans phases.
- Never force a poor match — when unsure, present candidates and ask.
- Respect the target skill's own gates (e.g. push/commit confirmations).
- Empty request → ask what they want, with 2-3 examples.
