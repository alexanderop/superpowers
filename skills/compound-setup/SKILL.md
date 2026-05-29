---
name: compound-setup
description: One-time initializer for a project's compound knowledge store. Scaffolds docs/solutions/ with a seed index.md, a README explaining the structure, and a discoverability note in CLAUDE.md/AGENTS.md so the SessionStart index-injection and auto-index hooks have something to work with from day one. Use once per repo when the user wants to "set up compound", "set up the knowledge store", "initialize docs/solutions", or "set up the brain".
argument-hint: ""
---

# Compound Setup — Initialize the Knowledge Store (run once per repo)

**Announce at start:** "I'm using the compound-setup skill to initialize this project's knowledge store."

## Overview

Run this **once** in a repo to create the `docs/solutions/` knowledge store that the `compound`
skill writes to and the `/lfg` pipeline reads from. After this, the plugin's hooks take over
automatically:

- `index-solutions` (PostToolUse) rebuilds `docs/solutions/index.md` whenever a solution file changes.
- `inject-solutions-index` (SessionStart) injects that index into every future session.

This skill seeds an initial `index.md` so injection works *before* the first solution doc exists,
and wires up discoverability. It is **idempotent** — safe to re-run; it never clobbers existing
docs or a populated index.

## Process

### Step 1 — Detect existing setup

```bash
ls docs/solutions/ 2>/dev/null
```

- If `docs/solutions/` already contains solution docs (any `.md` other than `index.md`/`README.md`),
  the store is already in use. Report that and STOP — do not overwrite anything. Offer `/compound`
  for adding docs instead.
- If it doesn't exist, or exists but is effectively empty, proceed.

### Step 2 — Create the directory and seed files

```bash
mkdir -p docs/solutions
```

Write **`docs/solutions/README.md`** (only if absent) explaining the store. Suggested content:

```markdown
# Documented Solutions

A searchable knowledge store of past problems and durable learnings. Each unit of engineering
work should make the next one easier — this is where that compounding happens.

- One file per problem/learning: `docs/solutions/<category>/<slug>.md`
- YAML frontmatter for search and dedup (see the `compound` skill's `references/schema.md`)
- Two tracks: **bug** (something broke and was fixed) and **knowledge** (a practice/decision worth keeping)
- `index.md` is auto-maintained by a hook — do not edit it by hand
- Add entries with the `compound` skill; this store is read automatically at session start
```

Write a **seed `docs/solutions/index.md`** (only if absent) so the SessionStart hook has something
to inject immediately:

```markdown
# Documented Solutions

Knowledge store of past problems and learnings. Empty so far — run the compound skill after
solving something to populate it. This index is rebuilt automatically when solution files change.
```

Do NOT pre-create empty category subdirectories — the `compound` skill creates the right category
directory on demand (`mkdir -p docs/solutions/<category>/`). The category list lives in the
`compound` skill's `references/categories.md`.

### Step 3 — Discoverability note

Ensure the project's substantive instruction file (`AGENTS.md` or `CLAUDE.md` — edit the one with
real content, not a shim that just `@`-includes the other) tells an agent the store exists, its
structure, and when it's relevant. If neither file exists, create the minimal one the project
conventionally uses (prefer `CLAUDE.md` for Claude Code projects).

Add the **smallest** natural addition — a single line in an existing architecture/directory section
beats a new heading. Keep the tone informational, not imperative:

```
docs/solutions/  # documented solutions to past problems (bugs + knowledge), by category, with YAML frontmatter (module, tags, problem_type) — relevant when implementing or debugging in documented areas
```

If the file already conveys this, make no edit.

### Step 4 — Report

Tell the user, concisely:

- What was created (`docs/solutions/`, `README.md`, seed `index.md`) and what already existed.
- Whether a discoverability line was added, and to which file.
- That the hooks now run automatically and they should use `/compound` (or `/lfg`, which calls it)
  to add learnings — and that this setup only needs to run once per repo.

Suggest committing the scaffold so it travels with the repo.

## Idempotency rules

| Situation | Action |
|-----------|--------|
| `docs/solutions/` has real solution docs | STOP, report already-set-up, change nothing |
| Seed `index.md` / `README.md` already exist | Leave them as-is |
| Discoverability note already present | Make no edit |
| Nothing exists | Create everything |
