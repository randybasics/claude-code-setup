---
name: update-memory-bank
description: Update the memory-bank docs to standard — read startHere first, route the change to the right file, keep startHere's map and sources-of-truth in sync, prune stale context, and verify before finishing. Use when "update memory bank", "add to memory bank", "document in memory-bank", "update activeContext", or after a significant change.
---

# Update Memory Bank — Keep the Shared Brain Current

## Purpose

The memory-bank is the context every agent reads first. A change that isn't written back leaves every
future agent working from a stale map. This skill routes an update to the right file, keeps
`startHere.md` and the `sources_of_truth` accurate, and prunes what's gone stale — so the map stays
trustworthy instead of quietly drifting.

## Protocol

### 1. Log usage

```bash
mkdir -p .claude/skills/.usage
printf '%s\t%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "update-memory-bank" >> .claude/skills/.usage/log.tsv
```

### 2. Load the framework first (mandatory)

Read [`memory-bank/startHere.md`](../../memory-bank/startHere.md) before touching anything — it's the
ranked map, the size guide, and the sources-of-truth. Then read whichever target file(s) the change
actually touches. Never edit the bank blind.

### 3. Route the change to the right file

| Change | Target file |
|---|---|
| What we're working on now | `activeContext.md` |
| A canonical fact (price, stack, config) | its source-of-truth file (e.g. `pricing.md`, `architecture.md`) — **not** just a mention |
| What the project is / who it's for | `projectbrief.md` |
| A new file or capability | create/point the file **and** add a ranked row to `startHere.md` |

### 4. Respect the size guide

🟢 **<400 lines** → edit directly · 🟡 **400–600** → navigate by section headers · 🔴 **>600** → edit
under the `## Executive Summary`, don't reload the whole file.

### 5. Apply the standards

Standard `## Section` headers · dated entries · working cross-links · each file stays in its lane
(activeContext = now; canonical facts live in their source file).

### 6. Sync the map + sources-of-truth (the dual-update)

- Added / moved / renamed a file or capability → update `startHere.md`'s ranked table **and** its
  `sources_of_truth` frontmatter.
- Touched a fact that has a source-of-truth file → update that **canonical** file; if any other file
  repeats the old value, fix or delete the stale mention (canonical wins).
- Bump `last_updated` in the frontmatter of every file you edited.

### 7. Prune stale context

`activeContext.md` is *current* only. Move shipped work out of "This sprint", drop anything no longer
active, and relocate durable detail into its canonical file. Keep it short — a sprawling activeContext
is the same problem as a stale one.

### 8. Verify (blocking) + report

Checklist: [ ] startHere read first · [ ] change landed in the right file · [ ] startHere map +
sources-of-truth in sync · [ ] canonical fact updated at its source, stale mentions fixed · [ ]
`last_updated` bumped · [ ] activeContext pruned · [ ] cross-links resolve.

Then report: files updated (what changed) · map / sources-of-truth synced (or N/A) · stale mentions
fixed · what was pruned.

## Rules

- Read `startHere.md` first — every time. It's the map; don't edit blind.
- One fact, one home: update the source-of-truth file, not a copy. A price lives in `pricing.md`.
- Keep `startHere.md` current — a stale map makes every agent stale.
- `activeContext.md` holds only what's active now; archive or relocate the rest.
