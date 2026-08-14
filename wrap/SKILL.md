---
name: wrap
description: End-of-session close-out — secure uncommitted work, put every open item in a durable home, refresh shared project state, and report honestly what was captured and what was skipped. Use when Sean is done for the day or longer ("wrap", "wrap it up", "done for now", "that's it for today", "end session", /wrap), including when the session contained almost nothing — it then correctly does almost nothing rather than not running. Retrospective only: never emits a resume/pickup prompt. Do NOT use for mid-task context recycling where the same work continues in a fresh thread (that is /wrap-continue), for answering questions about what wrapping does, or when a single narrower capture was asked for (just commit this, save one note).
license: MIT
metadata:
  version: 1.0.0
  category: workflow
  domain: session-management
---

# Wrap

Close out a session so nothing valuable dies with it, and so the last thing Sean reads is true.

**Route first.** Stopping → wrap. Continuing the same work in a fresh thread → `/wrap-continue`, stop here. The signal is stopping vs. continuing, not how much context was burned.

---

## Hard rules

These are the failure modes. Everything else is judgment.

1. **Freeze, then re-check.** Before anything is captured or claimed, stop background agents and background shells this session started (`TaskStop`; `TaskList` to find them). Leave dev servers and anything Sean wants running up — record port and PID so tomorrow doesn't start a second one. Then re-run `git status` in *every* worktree those processes touched. A killed process routinely leaves half-written files; that work is either committed on a clearly-labelled branch or named in the close-out as unverified. Never silent.

2. **Never push unasked.** Commit freely — commits are cheap and reversible. Pushing is outward-facing: offer it as a question and wait for the answer. Never force-push unless asked for by name. Same posture for anything irreversible or subjective — deleting entries, promoting a parked idea into active work, changing someone else's branch. One decision at a time, each with enough context to answer; not one batched "cut all eight?".

3. **Verify delegated work against the filesystem, not against the report.** Only applies if this thread delegated. A subagent's "done" is a claim: check that the file exists, the commit landed, the scope was met. A gap is either closed now or written into the durable open-item records — never softened, never reported resolved. This check runs *before* anything downstream is written on the strength of it.

4. **Dependency order.** freeze → verify delegations → commit code → tickets → reconcile open items → learnings → shared state → session record → docs commit → offer push. Artifacts cite identifiers that already exist. A session note cannot cite the SHA of the commit that contains it. Reconcile the backlog before writing shared state, so the two records don't contradict each other about what's open.

5. **One home per fact.** A learning lives in exactly one durable file and is referenced elsewhere by name — never restated in full inside `ORCHESTRATOR.md`, a session note, and a memory file. Same for the session's narrative: one long-term store, not two.

6. **Never invent.** If it isn't observable in the repo, the files, or this thread, you don't know it. No inferred prior incidents, no assumed delegation outcomes, no reconstructed history to make a sentence land harder.

7. **Retrospective only — no pickup prompt.** No "next session starts here", "picks up at", "next step is to…". Naming an item as open is required; instructions for resuming it belong to `/wrap-continue`. Asking Sean for a decision is not a pickup prompt and is welcome.

---

## Where things go

Match what's already in each file: edit the existing entry rather than appending a second one, keep its headings and schema, don't invent new sections to hold your output.

| Content | Home |
|---|---|
| Open / deferred / unfinished items | project-root `BACKLOG.md` (create if absent) |
| Current project state and pointers to it | `.claude/projects/<escaped-path>/memory/ORCHESTRATOR.md` — state and links only, never the durable fact itself |
| Durable decisions, learnings, corrections, gotchas, in-flight-work detail | a named file in the same `memory/` dir (`agent-*.md`, `reference_*`, `feedback_*`), indexed in its `MEMORY.md`, linked from `ORCHESTRATOR.md` by `[[wikilink]]` |
| Superseded `ORCHESTRATOR.md` content (prior PICKUP block, closed items) | `.claude/projects/<escaped-path>/memory/ORCHESTRATOR-log.md` |
| Cross-project status | `~/.claude/projects/project-facts.md` — edit the existing block; new blocks follow the schema at the top of that file, including default flags (a freshly shipped milestone is `promoted: no`) |
| Deferred-intent log | `~/.claude/projects/-Users-seansmith-Code/memory/tease-capture.md` — its own 30-day prune rule is documented inside it |
| Session narrative | the repo's existing notes location for code projects, **or** the project's Second Brain thread for cross-domain/life projects — one, never both |
| Tickets | Linear via `mcp__claude_ai_Linear__save_issue`, if the project tracks tickets |

Session-scoped facts never enter a cross-session file at all — not `ORCHESTRATOR.md`, not a memory file. Test before writing a line into either: would a fresh thread that never reads this line do something wrong because of it? A PID, a port, "the MCP was down this session" — none of these are evaluable later, so none of them get written, even as a status update.

Stage by explicit path. `git add -A` sweeps another session's edits when two threads share a tree.

---

## The pass

Read `~/.claude/BREVITY.md` before writing any notes or close-out prose — it sets the 150–350 word target range and structure rules the session record and close-out below must follow.

Run what the session earned. Steps 0 and 4 are the load-bearing ones.

0. **Freeze.** Rule 1. One line if nothing was running.
1. **Verify delegations.** Rule 3. Nothing to do if this thread delegated nothing — say that rather than omitting it.
2. **Commit code.** If the tree is clean or the diff is noise, skip and say so.
3. **Tickets.** Only if tracked tickets moved. Link the real SHAs from step 2.
4. **Reconcile open items.** Walk the thread's task list item by item: finished → marked finished, not carried forward as noise; still open → into `BACKLOG.md`, no duplicate of an item already there. **This runs whether or not the session produced any learnings** — it is not gated on step 5. It is the mechanism that makes "deferred is not dropped" true.
5. **Learnings.** Only non-obvious ones — corrections, decisions, conventions, gotchas that cost real time. Nothing re-derivable from the code or the git log. Write once, index it.
6. **Deferred-intent review.** Show what has accumulated in `tease-capture.md` since it was last reviewed, with each entry's age. Only raise archiving for entries 6+ months old (name them) — archiving means moving them to `tease-capture-log.md`, never deleting. Don't ask about entries younger than 6 months; that silence is the rule. Separately, offer promoting any entry that is a genuine actionable spike, at any age, one at a time. Nothing archived or promoted automatically. Skip if empty or already reviewed.
7. **Shared state.** Where the project has `ORCHESTRATOR.md`, updating it is **mandatory** if *any* of these are true: a subagent was delegated (success or failure), an architectural decision was made, in-flight work was added or completed, step 1 deferred a gap, the session produced commits, or project state changed in a way another thread would need. Skippable only when the file doesn't exist, or when *none* of those hold. Being an implementation thread rather than the orchestrator is not an excuse; neither is a clean ending.

   The update is not complete until something has been evicted, or you've confirmed nothing is stale and said so. Eviction, not addition, is the point of touching this file:
   - **PICKUP holds one block — the current one.** Writing a new block means the prior block moves to `ORCHESTRATOR-log.md` in the same edit, not alongside it.
   - **In-Flight holds genuinely open work only,** each item compressed to status + next action + pointer — not restated prose.
   - **Fragile Areas is the target shape for anything trap-like:** one line, a `[[wikilink]]` to the file that owns the full text, nothing restated in `ORCHESTRATOR.md` itself.
   - **Durable traps route to the file that owns them, never to `-log.md`.** Archiving a permanently-true fact is how it stops protecting anyone; `-log.md` is for what has actually closed or been superseded, not for things that are still true but merely old.
8. **Session record.** Ties the session to its real artifacts: commits made, tickets touched, memory files saved, gaps deferred and where they went. Skip for a session that produced none of those — an empty note committed to look thorough is worse than no note.
9. **Docs commit.** Everything steps 4–8 wrote gets its own commit, separate from the code commit, so documentation and code history stay distinguishable. Then offer the push (rule 2). Documentation left uncommitted is the wrap failing at its own job.

---

## Flagging open items

- An item carrying only a question, "needs input", "TBD", or a phase label — no concrete proposal to react to — gets flagged inline **decide or kill**, with a strawman attached when you can form one. A passive placeholder is how requested work silently becomes never-done.
- An item that has been carried before gets `(carried N×)` where N is the true count read from the record and incremented by one — not restated, not reset, not guessed.

## Size ceilings

Files that load automatically into every future session (`MEMORY.md`, `ORCHESTRATOR.md`, `CLAUDE.md`) cap at **20,000 characters, measured with `wc -c` — never in lines.** A line count hides real growth here: these files are written in paragraphs, so a file that blows the char cap 5x over can still read as "a few hundred lines." Keeping one current means cutting what has gone stale, not only appending what is new. If one is over, say so in the close-out and offer a specific prune naming the sections — never delete on your own authority. Leaving an oversized always-read file unmentioned is a miss.

## Proportion

- **Nothing happened** — a line or two, maybe a single saved note. The flow still ran; it just found nothing. Do not manufacture capture to fill it.
- **In a hurry** — secure the code, name what's open, stop. Then say which steps you skipped for time.
- **Big session** — run the steps its content triggered, not all of them by default.

A skipped step is a correct outcome. A skipped step that vanishes from the close-out is not.

---

## The close-out

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
```

Default to bullets, not paragraphs. **The banner above always leads the close-out** — it is a marker, not content, and does not count against the word target below. **No boxed status table, no per-step checklist** — those restate the close-out itself, and that repetition costs the read. Length scales with the session: two to five lines when little happened; for a heavy session, a short labelled list, one line per area, ~15 lines at the outside. 150–350 words is the target range per `~/.claude/BREVITY.md` — over range means reorder, deduplicate, or demote, never cut something load-bearing.

Lead with the thing Sean most needs to know — a gap, a broken state, a decision waiting on him — not a chronology.

Cover, in whatever form fits:

- commits, with SHAs, and whether they are pushed or still local
- anything left running or left broken, stated plainly (a branch that doesn't compile says so here, not only three files deep)
- where each open item landed
- what was skipped, and why
- what needs his decision

Every claim is something you checked *this turn*. "Unverified" is a perfectly good word. "Clean tree" is a claim — re-check it after your last write, or don't make it.

Shape, for a light session:

> Two commits, both local — `<sha>` (the fix) and `<sha>` (docs). Push when you want them off this machine.
> Nothing was running. The one unresolved question went to `BACKLOG.md`, flagged decide-or-kill.
> Skipped notes, memory and the idea log — nothing new to put in them.
