---
name: wrap-continue
description: Checkpoint a long-lived thread mid-task so it can be thrown away and resumed in a fresh one — stop background writers, commit, smart-push, reconcile open items into BACKLOG.md, and render a copyable pickup prompt inline. Use when the work is NOT finished and continues immediately in a new thread ("wrap and continue", "checkpoint", "context reset", "trim and keep going", "recycle this thread"). Do NOT use when the session is actually over — done for the day, work complete, or switching to unrelated work; that is /wrap's heavy retrospective. The distinguishing test is whether the task is finished, not whether the thread is ending.
license: MIT
metadata:
  version: 1.0.0
  category: workflow
  domain: session-management
---

# Wrap + Continue

A checkpoint, not a retrospective. It exists because the heavy end-of-session flow burns the exact context the user is trying to reclaim — so proportion is a requirement, not a preference.

## Misroute check (one line, then move)

If the signal is "done for the day" / "work is complete" / "switching to something else", this is the wrong flow. Say which flow applies and **then run that flow in the same turn** — do not stop and hand the user back a command to type; they already asked to wrap. A wrongly-run heavy flow costs context; a wrongly-run light flow loses capture. Name the signal that decided it in one sentence and get on with it. No pickup prompt in that case.

Everything below assumes the task continues.

Read `~/.claude/BREVITY.md` before writing any capture or pickup prompt — it sets the 150–350 word target range and structure rules that Section 3 and Section 5 below must follow.

## 1. Stop background writers first — ordering is load-bearing

Nothing may be observed about repository state until everything that could still be writing has stopped.

- List and stop background agents/shells (`TaskList` / `TaskStop`, plus background Bash this session launched).
- **Exception: leave long-running processes the next thread will want** — dev servers, booted simulators, watchers. Note the PID, port/UDID and the command so the next thread reuses it instead of starting a second one.
- **Then re-check `git status` in every location a stopped writer could have touched** — main tree *and* every worktree (`git worktree list`; worktrees live under `<repo>/.claude/worktrees/`), plus any other directory it wrote to. A clean root proves nothing about the trees.
- For each stopped agent, state: what it was doing, what it committed, what it left behind unfinished. Partial work is committed on a clearly labelled WIP commit (message names the breakage) **and** named in the pickup prompt as unverified — never left silent, never left to look finished.

> Reversing this order caused a real incident: a wrap reported a clean tree, the agents were killed after, and a half-built route died uncommitted under a false "nothing uncommitted anywhere" claim. A status read taken before the stop is not citable.

If nothing is running, one line and move on.

## 2. Commit, then push

**Committing is mandatory whenever anything is uncommitted. Never ask, never invent an exception** ("it's only a context reset" is not one — the thread's disk state is exactly what is at risk).

- Stage by explicit path, never `git add -A` — a concurrent session in the same tree gets swept in otherwise.
- If `git status` reports an ahead/behind count, `git fetch` before trusting it; a stale remote-tracking ref that disagrees with `git log` is a real and common trap. Re-check after pushing.

**Push is the default. Decide, never ask:**

| Case | Action |
|---|---|
| Nothing to push | say nothing at all |
| Remote named `upstream`, or `main`/`master`/`develop` on a remote other than the user's own default | skip, one-line note: `Push skipped — upstream/protected remote, push manually if intended` |
| Everything else — including `main` when it tracks `origin`, and deploy/staging remotes pushed to routinely | push |

Refusing an `origin` push on your own safety reasoning ("this deploys production", "want me to push?") is a failure, not caution. The narrow skips exist for one reason: never create an outward-facing effect resembling an upstream contribution (prior Ghostty incident — an unintended PR opened upstream).

## 3. Light capture — only what the code cannot tell you

Bar: *would this be unrecoverable if context cleared right now?* Corrections, decisions, constraints, gotchas. Skip freely; do not pad, do not restate rules that already live in `DESIGN.md` or an existing memory file.

- New correction → `.claude/projects/<path>/memory/feedback_*.md`; new durable reference → `reference_*.md`. One or two lines of genuine signal, in the file's existing format. If `MEMORY.md` is an index, add the pointer — an unindexed file is invisible.
- Project has an `ORCHESTRATOR.md`? Update its Decision Log / In-Flight Work — **regardless of thread type**. Implementation threads change project state too, and that state must be written back before context clears.
- `~/.claude/projects/project-facts.md` (cross-project status index) — if the file exists and this session changed something it tracks, update the project's block in place, matching the existing entry format. A shipped milestone gets a `Shipped: <date> — <what>. Promoted: no.` line; without the promotion state, downstream promotion/content workflows cannot find it. If the file does not exist, say nothing about it.

Under time pressure this section is the *only* thing that gets cut. Never the commit, the push, the ledger, or the pickup prompt.

## 4. Reconcile the ledger — always runs, even when capture is skipped

State the thread's **locked objective** in one line. Then walk every item in the task ledger:

- Done → marked complete, not carried.
- Open → recorded in the project-root `BACKLOG.md` (create if absent; append under a dated heading in the file's existing style; never duplicate an item already listed). Label each as *carried* or *parked* so a carried item does not read as duplicate parking. BACKLOG.md is the durable record that survives the wipe — the pickup prompt is lossy by design and must never be an item's only record.
- **Objective filter:** classify each open item as serving the locked objective or not. Off-objective items are parked and **excluded from "What's next"** — at most a one-line "parked in BACKLOG.md" pointer. Finished-but-off-objective work is closed out, not promoted into the next thread as a live decision. When in doubt, park: a wrongly parked item costs one line to restore; a wrongly carried one derails the restart.
- **Form check on carried items.** An item phrased as a question, "needs Sean's input", "TBD", or "awaiting a decision" is a defect signal, not a normal item. Either build the concrete first-pass strawman **now** (and tell the next thread to apply or redline it rather than re-ask), or surface it in the prompt as `DECIDE OR KILL:`. Producing the strawman is the stronger answer nearly every time.
- An item carried across a previous reset gets stamped inline in the prompt text: `carried 2× since 2026-08-01`. No separate tracking file for this — the stamp is the mechanism.

## 5. Pickup prompt

Always produced — it is the point of the exercise. Nothing uncommitted and nothing to capture is still a hit: the run is just short.

Compact — sized for a fast re-read, not a report. 150–350 words is the target range per `~/.claude/BREVITY.md` — over range means reorder, deduplicate, or demote, never cut something load-bearing. Default to bullets. It carries only:

- What is being built (one sentence).
- Where it stands, with the exact committed state (branch + SHA).
- The specific immediate next action — actionable, on-objective. "Continue the work" is a failure.
- Non-obvious context the codebase cannot supply: constraints, decisions, gotchas, conclusions already reached (say *don't re-derive X* — that carries the answer without the tokens that caused the reset).
- Anything a stopped agent left behind, named as **unverified**. Anything left running, with how to reuse it.
- `Working directory:` as an **absolute path** (`/Users/seansmith/Code/...`), not a tilde.

**Render it inline in the response, in a single fenced block, and put every heading, label and instruction to the user *outside* the fence.** Anything inside the fence gets pasted into the next thread; a stray `## Pickup prompt` heading or a slash command inside it lands as garbage there. That block is the primary deliverable — the user must be able to see it, scroll back, and re-copy it.

**Thread name.** `jq -r '[.nameSource, .name] | @tsv' ~/.claude/sessions/$CLAUDE_PID.json 2>/dev/null`. Carry it forward only if `nameSource` is `user`. If it is `derived`/`auto`, or the file or `jq` is missing, skip in silence — an auto slug like `code-55` carries no signal and must never be surfaced as though it did. When a user-set name exists: put `Thread: <name>` as the first line *inside* the block (prose, not a command), and below the block, outside it, give the working mechanism — `type /rename "<name>" in the new thread (or launch it as claude -n "<name>")`. `/rename` only fires as an entire message, so it cannot live in the paste.

**Clipboard** (convenience, best-effort, never a blocker):

```bash
cat <<'EOF' | pbcopy
[prompt content]
EOF
```

Unavailable (non-macOS, no clipboard)? One footnote in the status table. Not an error, not repeated in prose.

## 6. Output shape

A status table so the run can be confirmed at a glance, then the pickup prompt. Row vocabulary: Agents · Git · Capture · Backlog · Repo · Clipboard · Thread.

**Omit any row that does not apply this run** — no `None running`, no `No change`, no empty Thread row. A row that says nothing is noise.

```
  ┌──────────┬──────────────────────────────────────────────┐
  │ Agents   │ 2 stopped — settings-form left it broken     │
  │ Git      │ 3 commits — pushed to origin (abc1234)       │
  │ Repo     │ main clean · session-2 clean, WIP pushed     │
  └──────────┴──────────────────────────────────────────────┘
```

When more than one working tree was involved, report repo state **location by location**. A single "all clean" row reads at a glance as "everything is fine" and hides the tree that is broken.

**Proportion is scored.** Outside the fenced prompt, the response is the table plus at most a couple of lines the table cannot carry. Do not re-narrate the table in prose. Do not add ASCII art, gift-box graphics, banners, or scissor rules — unrequested decoration is a defect in a flow whose entire purpose is saving tokens. When there is genuinely nothing to do, a long response is itself the failure.

Whole flow: a handful of exchanges. No new tracking files, no cleanup, no scope beyond the one continuing task — `BACKLOG.md` is the single sanctioned addition.
