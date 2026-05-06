# CAW — Coordinated Agent Workflow

A reusable Claude Code agent template for delivering software changes through a planning → execution → review pipeline.

## What it gives you

Three agents and two slash commands:

- `/new-phase <description>` — `architect` writes a phase plan → `coordinator` reviews it and creates task specs.
- `/run-phase` — `executor` implements each task → `coordinator` reviews the code task-by-task → final phase sign-off.

The pipeline enforces an independent code review (the coordinator never sees the executor's narrative — only the task spec and the changed files), explicit business-vs-technical escalation, and a strict separation between phase plans, task specs, and reports.

## Install in a new project

In the target project's working directory, run Claude Code and tell it:

> Install CAW into this project. Read the install guide at `<path-or-URL>/INSTALL.md` and follow it step by step.

Where `<path-or-URL>` is either a local clone of this repo or a URL Claude can fetch. Claude will:

1. Confirm the destination directory.
2. Detect any pre-existing CAW files and ask whether to overwrite, skip, or abort.
3. Scan the repo to infer project name, stack, quality-gate commands, module rules, canonical domain docs, and any "phase" terminology collision — then show you a single confirmation message (in your language) so you can correct what it got wrong before anything is written.
4. Copy the agent and command templates into `.claude/`.
5. Generate a `CLAUDE.md` from the template, substituting in the confirmed values.
6. Scaffold `docs/workflow/` (`task_queue.md`, `planning/project_plan.md`, empty `tasks/` and `reports/` directories).

After install, run `/new-phase <description>` to plan the first phase, then `/run-phase` to execute it.

## Repo layout

| Path | Purpose |
|------|---------|
| [INSTALL.md](INSTALL.md) | Step-by-step prompt that Claude follows to install CAW into another project. |
| [templates/.claude/agents/](templates/.claude/agents/) | Agent definitions (`architect`, `coordinator`, `executor`) — copied verbatim. |
| [templates/.claude/commands/](templates/.claude/commands/) | Slash commands (`new-phase`, `run-phase`) — copied verbatim. |
| [templates/.claude/settings.json](templates/.claude/settings.json) | Default permissions skeleton. |
| [templates/CLAUDE.md.template](templates/CLAUDE.md.template) | `CLAUDE.md` skeleton with placeholders for project-specific data. |
| [templates/docs/workflow/](templates/docs/workflow/) | Workflow folder skeleton (`task_queue.md`, `planning/project_plan.md`). |
| [docs/caw-overview.md](docs/caw-overview.md) | How the CAW pipeline works end-to-end. |
| [docs/customization.md](docs/customization.md) | How to add specialized agents or override base behaviour per project. |
| [docs/extra-agents/](docs/extra-agents/) | Standalone examples of specialized agents you can drop into `.claude/agents/` *after* install if you decide you need them. Not part of the install flow. |

## Design intent

- **One install kit, identical behaviour everywhere.** Base agents (`architect`, `coordinator`, `executor`) and base commands (`/new-phase`, `/run-phase`) are project-agnostic. They never name a specific project; they always read project specifics from `CLAUDE.md`. This guarantees you get the same workflow shape in every repo.
- **Project specifics live in one place.** `CLAUDE.md` declares the project name, stack, quality-gate commands, architecture rules, canonical domain docs, and any extra specialized agents. Change `CLAUDE.md`, change the behaviour — without forking the agents.
- **Templates are copied verbatim.** The installer must not paraphrase or shorten the agent/command files. Drift between projects is the failure mode; that's why all customization happens in `CLAUDE.md` rather than in the agent prompts themselves.
