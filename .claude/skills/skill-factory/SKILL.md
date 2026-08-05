---
name: skill-factory
description: Scaffold a new agent skill to spec — valid frontmatter, clear triggers, tight protocol. Works across hosts (Claude Code, Codex, Antigravity). Use when "new skill", "scaffold a skill", "create a command", "make a skill for this".
---

# Skill Factory — Scaffold New Skills

## Purpose

Turn a repeatable workflow into a well-formed skill: correct frontmatter, sharp trigger phrases, and a
protocol tight enough that the agent actually follows it. Repeatable work should become a skill, not
a re-typed prompt. Write the protocol host-neutral by default — don't bake in a specific invocation
syntax (`/name`, `$name`, or "mention by name") since that's decided by whichever host loads the file.

## When to Activate

- "new skill", "scaffold a skill", "create a slash command", "turn this into a skill"

## Protocol

### 1. Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "skill-factory" >> .claude/skills/.usage/log.tsv
```

### 2. Define the job in one sentence

What does this skill do, and when should it fire? If you can't say it in one sentence, it's two skills.

### 3. Write the frontmatter

```markdown
---
name: kebab-case-name
description: <what it does> + <the trigger phrases a user would actually say>. Use when "...", "...".
---
```

- `name`: kebab-case, matches the directory.
- `description`: this is what the router reads to auto-activate. Pack in the real trigger phrases —
  the words a user would actually type — not a vague summary.
- `name` + `description` are the two fields every host (Claude Code, Codex, Antigravity) understands.
  Anything else (e.g. `disable-model-invocation`) is Claude-Code-specific — fine to add, but don't
  depend on it for the skill's core behavior since other hosts will just ignore it.

### 4. Write the protocol

- Step 1 is always usage logging, exactly like Step 1 of this skill but with `<skill-name>` swapped
  for the new skill's own `name`:

  ```markdown
  ### 1. Log usage

  ​```bash
  mkdir -p .claude/skills/.usage
  printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "<skill-name>" >> .claude/skills/.usage/log.tsv
  ​```
  ```

  The skill's own logic starts at Step 2. Applies to skills scaffolded from scratch and to existing
  skills being updated — if a skill you're touching doesn't have this step yet, add it as its new
  Step 1 and renumber the rest.
- Number the steps. One action per step.
- Put the decision rules in tables, not prose.
- State the hard rules at the bottom ("always X", "never Y").
- Keep it as short as it can be while still unambiguous. Long skills get skimmed.
- To read usage back: count per skill with
  `awk -F'\t' '{c[$2]++} END{for (s in c) print c[s], s}' .claude/skills/.usage/log.tsv | sort -rn`;
  history for one skill with `grep -P '\t<skill-name>$' .claude/skills/.usage/log.tsv`.

### 5. Check for collisions

Does an existing skill already fire on these phrases? If so, add one line to your routing notes
clarifying which skill wins for the overlapping phrase, and why.

### 6. Place the file

```
.claude/skills/<name>/SKILL.md
```

`.claude/skills` is this repo's canonical source. `.agents/skills` is a symlink to it (`.agents/skills
-> ../.claude/skills`), which is the neutral path Codex and Antigravity actually discover skills
from — so a file placed here needs no separate copy to be picked up by either. Reference helper
files relatively from the skill directory if the protocol needs them.

## Rules

- Every skill (including this one) includes the Step 1 usage-logging block, unmodified except for
  `<skill-name>`.
- One concern per skill. If it has two unrelated trigger sets, split it.
- The `description` is the product — most skills fail because the triggers are vague, not because the
  body is wrong.
- Don't duplicate trigger phrases across skills; resolve the collision explicitly.
