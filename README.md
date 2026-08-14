> **This repo is archived. These skills moved to [seansmithworks/agent-skills](https://github.com/seansmithworks/agent-skills).**
>
> Install any of them directly:
>
> ```bash
> npx skills add seansmithworks/agent-skills --list
> npx skills add seansmithworks/agent-skills --skill wrap
> ```

# claude-skills

A collection of Claude Code skills for managing AI-assisted development workflows.

## Status

These skills are **early work-in-progress** — actively iterated on, currently used by one person, and not battle-tested across diverse setups. Versions are intentionally pre-1.0.

If you try them: expect rough edges, expect things to change, and review what each skill does before invoking — they run shell commands, modify your `~/.claude` setup, and interact with git. **No warranty. Use at your own risk.**

## What's here

Skills are loaded by Claude Code from `~/.claude/skills/`. Each skill is a directory containing a `SKILL.md` file that Claude reads when you invoke the corresponding slash command.

### wrap

End-of-session wrap-up. Commits outstanding work, generates session notes, updates orchestrator state, captures memories, and produces a hot-resume pickup prompt. Use when done for the day (or longer).

```
/wrap
```

### wrap-continue

Mid-task context reset. Lighter version of wrap — smart push, hot-resume pickup prompt rendered inline. Use when recycling a long-lived thread to free context while continuing the same work.

```
/wrap-continue
```

### orchestrator-scaffold

Scaffolds missing project files for the Orchestrator pattern — either v2.1 lifecycle agent files (six domain specialists) or v2.2 grounding files (mission.md, brand.md, principles.md). Three modes: auto-detect, lifecycle, grounding.

```
/orchestrator:scaffold
/orchestrator:scaffold lifecycle
/orchestrator:scaffold grounding
```

### orchestrator-update

Reviews and applies pending updates to the orchestrator's config. Runs before/after evals to catch regressions. Opt-in — nothing auto-applies.

```
/orchestrator:update
```

## Installation

Use the `npx skills add seansmithworks/agent-skills` commands above to install any skill directly.

## Context

These skills are part of the [Orchestrator pattern](https://github.com/seansmithworks/claude-skills) — a long-lived Claude Code thread that learns your codebase, maintains context, and delegates implementation to subagents. The `orchestrator-scaffold` and `orchestrator-update` skills support that workflow.

`wrap` and `wrap-continue` are session management utilities usable independently of the Orchestrator pattern.

## License

MIT
