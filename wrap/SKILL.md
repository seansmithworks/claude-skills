---
name: wrap
description: End-of-session wrap-up — heavy capture, retrospective only. Use when done for the day (or longer). Runs reconciliation, commits, Linear, memories, orchestrator state, session notes, Second Brain. No pickup prompt — use /wrap-continue for that.
license: MIT
metadata:
  version: 0.6.0
  category: workflow
  domain: session-management
  status: stable
  platforms: All
keywords:
  - wrap
  - session
  - end
  - ship
  - done
  - cleanup
  - notes
  - linear
  - commit
  - push
---

# Wrap Session

End-of-session wrap-up. Heavy capture. Use when the session is finished and the next pickup may be hours or days away.

Triggered by: `/wrap`, "wrap it up", "wrap", "wrap up", "done for now", "end session".

If you're recycling context mid-task and continuing the same work, use `/wrap-continue` instead.

Suggests what's worth saving before closing. Every step is optional — assess what's relevant and skip what isn't. Don't force a push or commit if nothing meaningful changed.

## Suggested Steps

The sequence follows the Information Lifecycle (L1 → L2 → L3): reconcile first (orchestrator threads only), then code so commits have SHAs, then hot memory, then the session log, then long-term threads. Each layer references the one below it.

### 1. Plan Reconciliation (orchestrator threads only)

Verify every subagent delegated this session delivered what was asked. Run BEFORE commits — the reconciliation result shapes everything downstream.

**How to verify:**

- `git log` for recent commits → confirm each delegation produced a commit (and a push)
- Re-read each delegation prompt → check "Verification" and "After Completion" sections. Did the subagent run tests? Commit? Push?
- Compare against plan or ticket scope → any requirement silently dropped?
- Check for uncommitted work in the tree — subagents sometimes leave staged/unstaged changes

**Two paths when a gap is found:**

- **Fix now** — small gaps (missing commit, unstaged file, one failing test): spawn a focused follow-up subagent. Wait for it to finish, then continue the wrap.
- **Defer** — larger gaps (missing feature, broken behavior, architectural rework): add to `ORCHESTRATOR.md` under **In-Flight Work**. Flag it in the final summary.

Skip entirely on non-orchestrator threads.

### 2. Commit & Push code (if dirty)

Check `git status`. If there are uncommitted code changes worth keeping, suggest a commit + push. If the tree is clean or changes are trivial, skip.

First code step so every downstream artifact (Linear, session notes, Second Brain) can reference real commit SHAs.

### 3. Linear / Ticket Tracker (if tickets were worked)

If the session completed or changed the status of any tracked tickets, suggest updating them. Check MEMORY.md or CLAUDE.md for the project's issue tracker. Match commits against known ticket IDs. For Linear, use `mcp__claude_ai_Linear__save_issue` — link commit SHAs from step 2.

Skip if the project doesn't use a ticket tracker or no tickets were relevant.

### 4. Memories (L1 — if new learnings surfaced)

Save feedback, decisions, or references that future sessions should know about. Don't save things derivable from code or git history. Update `MEMORY.md` index for new entries.

Done before session notes so notes can reference saved memory files. Skip if nothing non-obvious was learned.

**Ledger reconciliation (save open items forward, mark done items complete) — run this regardless of whether new learnings surfaced above; it's not optional.** Walk the thread's task ledger (the harness Task tools) item by item, before the Orchestrator State update in step 6:

- Items that are complete → confirm they're marked completed in the ledger. Don't carry a finished item forward.
- Items still open / not-done / deferred → save each to the project-root `BACKLOG.md` (create it if absent; append under a dated heading; don't duplicate an item already listed there).

BACKLOG.md is the durable carrier that survives `/clear` and `/compact` — any pickup prompt or compaction summary is lossy and must never be the only record of an open item. Running this before step 6 keeps ORCHESTRATOR.md's In-Flight Work and BACKLOG.md in agreement.

### 5. Idea-Log Review (tease-capture)

Read `~/.claude/projects/-Users-seansmith-Code/memory/tease-capture.md`.

Surface any entries that are new since the last wrap review (the most recent captures at the bottom of the Captures list), so Sean sees what accumulated since he last looked.

Then offer two things, one at a time:

1. **Prune pass** -- flag entries older than ~30 days that have not been acted on (the file's own documented rule). List them and ask Sean whether to remove them. Do not auto-delete.
2. **Promotion pass** -- for any entry that is a genuine, actionable engineering or design spike (not a content musing, strategy note, or explicitly-parked item), offer to promote it into the relevant project's `todos/` or backlog. One entry at a time, Sean's call. Do not move anything automatically.

This is a review-and-offer step, not an automatic drain. The log spans projects, so keep the review light -- it is one of several optional wrap steps.

Skip if the idea log is empty or all entries were already reviewed in a prior wrap.

### 6. Orchestrator State (any thread in a project that has an ORCHESTRATOR.md)

First, check whether `.claude/projects/<path>/memory/ORCHESTRATOR.md` exists for the current project. If it does, this step applies — regardless of whether this thread is the orchestrator or an implementation thread.

Update `ORCHESTRATOR.md` with architecture changes, decisions, fragile areas, recent delegations, and any in-flight work added or completed. Keep under 300 lines. Prune stale Decision Log and Recent Delegations entries.

If step 1 deferred any gaps as follow-up tasks, record them in **In-Flight Work** here.

**In-flight form-check:** Any In-Flight item that is gated on input (phrased as a question, "needs Sean," "Phase N," "TBD") rather than carrying a concrete strawman should be flagged inline as "decide or kill." Any item carried forward from a prior session without resolution should be stamped `carried N× since [YYYY-MM-DD]` so repeat-carries are visible. The primary home for this check is `/wrap-continue` Step 3c — apply it here too when writing or updating In-Flight Work.

**This update is mandatory if the project has an ORCHESTRATOR.md AND ANY of the following are true:**

- Any subagent was delegated this session (whether it succeeded or failed)
- Any architectural decision was made
- Any in-flight work was added or completed
- Step 1 deferred any gaps (they must land in ORCHESTRATOR.md)
- The session produced commits (code shipped that should be reflected in context)
- Project state changed in a way the orchestrator would need to know (role status, interview outcome, file locations, new conventions)

Skip ONLY if the project has no ORCHESTRATOR.md, AND no subagents were delegated, AND no decisions were made, AND no in-flight work changed.

> **Gotcha:** Implementation threads (non-orchestrator) often change project state — a CV generated, a role applied, a convention locked. These changes are exactly what goes stale if not written back. Don't skip just because you're not the orchestrator thread.
>
> **Gotcha:** A session that ends cleanly after shipping a milestone still needs ORCHESTRATOR.md updated — clean endings are not an excuse to skip. The skip is only valid for pure-research or pure-chat sessions where nothing changed.

### 7. Session Notes (L2 — if meaningful work happened)

Append to `docs/SESSION_NOTES.md` if the session produced commits, decisions, or learnings worth recording. Include commit SHAs from step 2, ticket IDs touched from step 3, references to memory files saved in step 4, and any deferred gaps from step 1.

Quick chat sessions or minor tweaks don't need notes.

### 8. Second Brain (L3 — if project state changed significantly)

Update the project's thread in Second Brain with current state, what's next, open questions. Use `/secondbrain` or QMD to find/update the thread. This is the narrative arc — summarize the session notes from step 7, don't duplicate them.

Skip for small sessions that don't change the project's trajectory.

**Don't run Session Notes AND Second Brain for the same session.** Pick one:

- **Coding-only projects** → Session Notes (git-searchable, co-located with code)
- **Cross-domain or life projects** → Second Brain thread (QMD-searchable, broader context)

### 9. Final Commit & Push (if steps above created docs)

Commit any session notes, memory updates, orchestrator state, or Second Brain changes created in steps 4–8. Push. Separate commit from step 2 so code and documentation commits are distinct in history.

## Output Format

After completing selected steps, present the wrap-it-up box and summary table.

```
                                               _____
                                              |     |
  ╭───────────────────────────────────────────[_____]───────────────────────────────────────────╮
  │                             ┌────────────────────────────────────────────────────────────┐  │
  │                             │                                                            │  │
  │   · · · · · · · · · · · ·   │                                                            │  │
  │   · · · · · · · · · · · ·   │               ________ ______ _______ ______               │  │
  │  · ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ·  │              |  |  |  |   __ \   _   |   __ \              │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │              |  |  |  |      <       |    __/              │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │              |________|___|__|___|___|___|                 │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │                                                            │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │                      _______ _______                       │  │
  │  ·◉ ○ ○ ○ ○ ○ ● ○ ○ ○ ○ ◉·  │                     |_     _|_     _|                      │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │                      _|   |_  |   |                        │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │                     |_______| |___|                        │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │                                                            │  │
  │  ·◉ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ◉·  │                       _______ ______                       │  │
  │  · ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ◉ ·  │                      |   |   |   __ \                      │  │
  │   · · · · · · · · · · · ·   │                      |   |   |    __/                      │  │
  │   · · · · · · · · · · · ·   │                      |_______|___|                         │  │
  │                             │                                                            │  │
  │                             │                                                            │  │
  │                             └────────────────────────────────────────────────────────────┘  │
  ╰─────────────────────────────────────────────────────────────────────────────────────────────╯


  ┌────────────────┬──────────────────────────────────────────────────────────────────────────┐
  │ Area           │ Status                                                                   │
  ├────────────────┼──────────────────────────────────────────────────────────────────────────┤
  │ Reconciliation │ 4/4 delegations verified; 1 deferred → ORCHESTRATOR.md                   │
  │ Git            │ 3 commits pushed (abc1234..def5678)                                      │
  │ Linear         │ PRJ-12, PRJ-15 → Done                                                    │
  │ Memories       │ 1 feedback saved                                                         │
  │ Idea Log       │ 3 new captures surfaced; 1 entry flagged for pruning                     │
  │ Orchestrator   │ Updated                                                                  │
  │ Session Notes  │ Updated                                                                  │
  │ Second Brain   │ Skipped — captured in session notes                                      │
  │ Repo           │ Clean, up to date with origin                                            │
  └────────────────┴──────────────────────────────────────────────────────────────────────────┘
```

The table sits inside the same fenced block as the box so both render as monospace and share a single character grid. Pad every Status cell with trailing spaces so the closing `│` lands at the exact same column as the box's right edge.

Adapt rows to what's relevant. Skip rows for steps that didn't run. The Reconciliation row only appears on orchestrator threads.

## Judgment Calls

- **Short session, no code changes?** Skip everything except maybe a memory if something useful came up.
- **Big session, lots of code?** Run reconciliation (if orchestrator), commits, plus whichever capture artifacts are load-bearing. Don't run every step just because the session was long.
- **Exploratory session, no conclusions?** Maybe just session notes for context if you'll pick it up later.
- **User seems in a hurry?** Commit + push only, skip the rest.
- **Orchestrator thread, gap found in reconciliation?** Fix-now for small gaps. Defer to ORCHESTRATOR.md for larger gaps. Never silently skip a gap.
- **Orchestrator thread, everything clean?** Reconciliation row should say "N/N delegations verified."
- Don't commit empty or trivial session notes just to check a box.
- Don't force-push or push to branches without asking.
