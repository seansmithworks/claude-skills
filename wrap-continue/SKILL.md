---
name: wrap-continue
description: Mid-task context reset — light capture, smart push, hot-resume pickup prompt rendered inline. Use when recycling a long-lived thread to free context while continuing the same work. NOT for end-of-day wrap — use /wrap for that.
license: MIT
metadata:
  version: 0.7.0
  category: workflow
  domain: session-management
  status: stable
  platforms: All
keywords:
  - wrap
  - continue
  - context
  - reset
  - checkpoint
  - token
  - compact
  - resume
---

# Wrap + Continue

Mid-task context reset. Light capture. Use when the session is being recycled to free context — the work is NOT done, you're resuming immediately in a fresh thread.

Triggered by: `/wrap-continue`, "wrap continue", "wrap and continue", "checkpoint", "context reset", "trim and continue".

If the session is actually done for the day, use `/wrap` instead — that runs full retrospective capture.

## Steps

### 1. Commit & Push (mandatory commit, smart push)

Check `git status`. Everything uncommitted is at risk when context clears. Commit all meaningful changes now — this is not optional.

If nothing is uncommitted, note it and move on.

**Push behavior — default ON with two smart skips, no prompt:**

- **Skip silently** if there is nothing to push (working tree already matches remote HEAD).
- **Skip with a one-line note** if the tracked remote is named `upstream`, or the branch is `main`, `master`, or `develop` on a non-`origin` remote. Status table note: `Push skipped — upstream/protected remote, push manually if intended`.
- **Otherwise push to origin.** No confirmation needed.

_Why these skips exist: commit protects against thread crash; push protects against disk failure and machine switches. Skipping upstream/protected remotes avoids accidental CI triggers and the upstream-PR landmine (prior Ghostty incident)._

### 2. Light Capture (optional — only if non-obvious)

Save feedback, corrections, or decisions that surfaced this session and aren't yet in memory. Keep this short. The bar is: "would I lose this if context cleared right now?"

- New feedback → save to `.claude/projects/<path>/memory/feedback_*.md`
- New reference → save to `reference_*.md`
- Major architectural decision, project state change, or in-flight work update → update `ORCHESTRATOR.md` Decision Log and In-Flight Work sections. This applies to ANY thread in a project that has an `ORCHESTRATOR.md` — not just orchestrator threads. Implementation threads change state too (CVs generated, roles applied, conventions locked) and that state must be written back before context clears.

Skip if nothing non-obvious came up. Don't run full retrospective — that's `/wrap`.

**2a. Ledger reconciliation (save open items forward, mark done items complete) — always runs, independent of whether the rest of Light Capture is skipped.**

Before generating the pickup prompt, walk the thread's task ledger (the harness Task tools) item by item:

- Items that are complete → confirm they're marked completed in the ledger. Don't carry a finished item forward.
- Items still open / not-done / deferred → save each to the project-root `BACKLOG.md` (create it if absent; append under a dated heading; don't duplicate an item already listed there).

**Objective-relevance filter (guards against drift across the reset):** State the thread's locked objective in one line first. Then classify every open item as either *serves-the-objective* or *off-objective* (a leftover from a tangent, or work that belongs to a different objective). Off-objective items go to `BACKLOG.md` and are **excluded from the pickup prompt's "What's next"** — they must never resurface as an active next-step in the next thread. Only objective-serving items are eligible to carry into the pickup prompt. When in doubt whether an item serves the locked objective, park it rather than carry it: a wrongly-parked item is one line to re-add, a wrongly-carried item derails the restart.

BACKLOG.md is the durable carrier that survives `/clear` and `/compact` — the pickup prompt itself is lossy and must never be the only record of an open item. This feeds 3c below: the items saved to BACKLOG.md here are exactly the carried items that get form-checked before the pickup prompt is written.

### 2b. Project Facts (one-line check)

Did project state change this session (status change, milestone shipped, something postable)? If yes, update the relevant block in `~/.claude/projects/project-facts.md` in place (set `promoted: no` on a new milestone). If no, skip — don't force it.

### 3. Hot-Resume Pickup Prompt

Generate a compact pickup prompt the user can paste at the start of the next thread to restore context exactly where work left off.

The pickup prompt must include:

- **What we're building** — one sentence on the project + task
- **Where we are** — current status, what just shipped (with commit SHA if applicable)
- **What's next** — the immediate next action (specific, not vague), scoped strictly to the locked objective. Only items that passed the objective-relevance filter in 2a belong here. Off-objective work is not a "next action" — it lives in BACKLOG.md and, if worth mentioning at all, appears only as a one-line "parked in BACKLOG.md" pointer, never as an active step.
- **Key context** — anything non-obvious that the next thread won't know from the codebase (design decisions, constraints, open questions, gotchas)
- **Working directory** — the repo path so the next thread can orient immediately

**3a. Render the pickup prompt inline** as a fenced code block between the cut lines shown in the Output Format below. This is the primary deliverable — the user must be able to see it, scroll to it, and re-copy from it. Do not skip this step.

**3b. Then also copy to clipboard** via `pbcopy`:

**3c. Carried-item form-check (run before finalizing the pickup prompt):**

Before writing the "What's next" section, scan every item you are carrying forward. For each item, assess its form:

- **Concrete strawman form** — item describes something to build or a specific action. Carry forward normally.
- **Gated-on-input form** — item is phrased as a question, "needs Sean's input", "Phase N", "TBD", or "awaiting decision" without a concrete strawman attached. This is a DROP SIGNAL, not a normal backlog item. Do one of two things:
  - Build the first-pass strawman now, before writing the pickup prompt, OR
  - Surface it loudly in the pickup prompt: "carried as a question — decide or kill."

For any item carried forward again (not resolved since the last wrap-continue), stamp it inline: `carried N× since [YYYY-MM-DD]`. This tag lives in the pickup prompt text itself — do not create any separate tracking file. Repeat-carries become visible so they can be decided or killed.

The goal: no gated item should quietly survive another cycle as a question. Either it becomes a strawman or it becomes an explicit decision point.

**3d. Thread name carry-forward.** Read the current session's user-set name:

```bash
jq -r '[.nameSource, .name] | @tsv' ~/.claude/sessions/$CLAUDE_PID.json 2>/dev/null
```

If `nameSource` is `user`, the name is meaningful and carries forward. If it's `derived` or `auto`, or the file/`jq` is unavailable, skip silently — a machine-generated slug like `code-55` carries no signal and must not be surfaced.

When a user-set name exists, do two things: add `Thread: <name>` as the first line inside the pickup prompt, so the next thread knows the working title even if Sean never renames it; and emit a `/rename "<name>"` line outside and below the fenced pickup-prompt block (see Output Format). The `/rename` line lives outside the fence because `/rename` is a built-in CLI command that only fires when it is the entire message — a paste that begins with it would land as literal text. It must be typed separately, not pasted as part of the prompt.

```bash
cat <<'EOF' | pbcopy
[prompt content here]
EOF
```

The inline render is the source of truth. The clipboard is a convenience copy of the same content.

If `pbcopy` is unavailable (non-macOS), skip silently — add a note in the status table that clipboard copy was unavailable.

## Output Format

```
  ╭─────────────────────────────────────────────────────────╮
  │  ↻  WRAP + CONTINUE                                     │
  ╰─────────────────────────────────────────────────────────╯

  ┌──────────┬──────────────────────────────────────────────┐
  │ Git      │ 2 commits — pushed to origin (abc1234)       │
  │ Capture  │ 1 feedback saved / skipped                   │
  │ Clipboard│ Copied to clipboard                          │
  │ Repo     │ Clean, up to date with origin                │
  │ Thread   │ adyen-case-study — carried forward           │
  └──────────┴──────────────────────────────────────────────┘
```

The `Thread` row is omitted entirely when there is no user-set name to carry.

Then the wrapped-present, cut-lines, and the pickup prompt rendered inline:

```
              ╱╲ ╱╲
             ╱  ╳  ╲
        ╭───────┴───────╮
        │       │       │
        │ ──────┼────── │
        │       │       │
        │  DO NOT OPEN  │
        │  UNTIL NEXT   │
        │   SESSION     │
        ╰───────────────╯

   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
```

```
## Pickup prompt (paste at start of next thread)

Thread: [name]
[project + task context]
[current status + last commit SHA]
[immediate next action]
[non-obvious context / constraints]
Working directory: ~/Code/[project]
```

```
   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ✂
```

**Pickup prompt is on your clipboard.** Paste it at the start of the next thread.

**Thread name:** type `/rename "adyen-case-study"` in the new thread (or launch it as `claude -n "adyen-case-study"`). Omitted entirely when there is no user-set name.

## Judgment Calls

- **Nothing uncommitted?** Still generate the pickup prompt — that's the whole point.
- **Nothing to push?** Skip push silently. Still commit if anything was uncommitted.
- **Upstream remote detected?** Skip push with the one-line note. Don't ask.
- **Big mid-session correction?** Save it as feedback before clearing — future threads will thank you.
- **Open item that isn't part of this objective?** Park it in BACKLOG.md — do not thread it into the pickup prompt. The hot-resume is for the locked objective only; an unrelated item crossing the reset is the exact drift failure this guards against.
- **Project has ORCHESTRATOR.md?** Update it — even if this is an implementation thread, not the orchestrator. Any state that changed this session (decisions, conventions, role status, files created) goes in Decision Log or In-Flight Work before context clears.
- **User seems in a hurry?** Commit + push (if applicable) + pickup prompt only. Skip capture.
- **Session name is `derived`/`auto`, or the sessions file is missing?** Skip the whole thread-name carry-forward step silently — never surface a machine-generated slug.
- Keep the whole flow under 5 exchanges. This is speed work, not ceremony.

## Future Setup Path (deferred)

If a second user adopts this skill, add a config file at `~/.claude/skills/wrap-continue/config.json` with: (a) origin remote name override, (b) custom protected branch patterns, (c) opt-out of auto-push. Not built now — premature for a one-user skill.
