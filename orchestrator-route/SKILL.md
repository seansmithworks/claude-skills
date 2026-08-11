---
name: orchestrator-route
description: Route an over-cap ORCHESTRATOR.md down to a router-shaped file without destroying durable value. Use when a size gate fires, when /wrap reports the file is over its 20,000-char cap, or when Sean says "route this", "prune orchestrator", "this file is too big", "/orchestrator-route". NOT for a file that is merely untidy — the trigger is size or staleness, and the operation is routing facts to their owners, never deleting by age.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent
version: 0.1.0
---

# /orchestrator-route

`ORCHESTRATOR.md` is a **router**: current state plus pointers. It is not a store and not a log. When it is over cap it is because durable facts landed in it instead of in the file that owns them. The fix is routing, not pruning.

Everything below is a boundary. The procedure is obvious and you already know it; these are the six things that get it wrong.

## ⛔ Gate 1 — Never evict by recency

**This is the one that destroys files.** The instinct is "keep the N newest." That rule inverts value: a trap from two months ago is still true, and a status line from yesterday expires next week.

Measured in seansmithdesign.com's 112K file: ~94 lines carry permanently-true traps, ~13 were dead the moment they were written. Deleting every worthless line moves 112,472 → ~103,000. **Deletion is not the lever.** Age is not the signal.

Sort by lifespan, never by date:

| Would a fresh thread do something wrong without this line? | Destination |
|---|---|
| No — and it can't even be evaluated now (`PID 86955`, "MCP was down this session") | **delete outright** |
| No — it is what happened | `ORCHESTRATOR-log.md` |
| Yes — permanently true about the project | the owning memory file (Gate 2) |
| Yes — and it is about right now | stays in `ORCHESTRATOR.md` |

## ⛔ Gate 2 — Route to the on-demand layer, not to `agent-*.md`

`orchestrator-boot/SKILL.md:53` reads **all `agent-*.md`** at boot. Moving traps from `ORCHESTRATOR.md` into `agent-craft.md` moves bytes from one always-read file into another. Net zero. The headline number drops and nothing is saved.

Durable traps go to `reference_*.md` / `feedback_*.md` in the same `memory/` dir — the on-demand tier — indexed in `MEMORY.md`, linked from `ORCHESTRATOR.md` by `[[wikilink]]`.

Use `agent-*.md` only for what a builder in that domain needs *every* time, not for incident traps.

## ⛔ Gate 3 — Durable traps never go to `-log.md`

`ORCHESTRATOR-log.md` is for what **closed or was superseded**. Archiving a permanently-true fact is how it stops protecting anyone — it is deletion with extra steps, and it feels safe, which is why it happens.

If it is still true, it gets a home and a pointer. If it is over, it gets archived.

## ⛔ Gate 4 — No fact may lose its home

Two archetypes. Check which one you have **before** planning the edit:

```bash
grep -o '\[\[[^]]*\]\]' ORCHESTRATOR.md | sort -u | wc -l
```

- **Routed-but-duplicated** (seansmithdesign.com: 75 unique wikilinks, all resolving). The destinations already exist; the prose is restated *around* the pointers. This is a **compression** pass. Gate: the resolving-wikilink count must not drop.
- **Never-routed** (jefe: 0 wikilinks, 19 dated session sections back to May). There are no destinations. This is an **extraction** pass — you must create the `reference_*`/`feedback_*` files and index them in `MEMORY.md`. Gate: every durable fact you removed appears in a named file you can `grep`.

Verify by grep after the edit, not by reading the diff. A trap silently dropped in transit is invisible in a 40K diff.

## ⛔ Gate 5 — Commit a baseline first

`git add <the file> && git commit` before any edit. There is no undo otherwise, and this is the highest-value state file in the project.

**Then diff before you stage.** `git add <path>` takes every hunk in that path including a sibling session's uncommitted work. Run `git diff <path>` first and commit only your own hunks. A clean `git status` showing one file is not evidence the commit holds only your work.

## ⛔ Gate 6 — Success is the whole boot set, not one file

Measure before and after:

```bash
cat MEMORY.md mission.md general-agent-context.md agent-*.md ORCHESTRATOR.md | wc -c
```

`ORCHESTRATOR.md` going down while the total stays flat means you moved bytes between always-read files (Gate 2 violation). Report both numbers or the run is unverified.

## The target shape

It already exists and works. In seansmithdesign.com, **Fragile Areas** holds 13 durable traps in 4,213 chars — one line each, a `[[wikilink]]` to the owner, zero restated prose. **PICKUP** holds the same wikilinks *plus* the full prose around them, and is 12× larger.

Point every trap-like section at the Fragile Areas shape.

Section rules after routing:

- **PICKUP** — exactly one block, the current one. Prior blocks move to `-log.md` in the same edit.
- **In-Flight** — genuinely open work only, each item `status · next action · pointer`.
- **Decision Log** — 3 newest *that are still live*; superseded ones to `-log.md`, durable lessons extracted to their own file first (Gate 3).
- **Architecture / Conventions** — rewritten, never appended.

## Scope

Run this **inside the target project's repo**, never from `~/Code`. The file is per-project state and a sibling thread may hold it — check whether `ORCHESTRATOR.md`'s mtime or HEAD moved since you started before writing.

One file per run. Do not batch projects.

## Done checklist

- [ ] Baseline committed, own hunks only
- [ ] Archetype identified (wikilink count run)
- [ ] Every durable fact greppable in a named file
- [ ] Wikilink count did not drop (compression) OR new files exist and are indexed in `MEMORY.md` (extraction)
- [ ] Boot-set total reported before AND after
- [ ] Nothing evicted by date
