# CAW Overview

CAW (Coordinated Agent Workflow) is a small, opinionated pipeline for delivering software changes through three Claude Code agents and two slash commands. This document explains how the pieces fit together.

## The pipeline

```
/new-phase <description>
        │
        ▼
   ┌───────────┐        ┌─────────────┐
   │ architect │ ──────▶│ phaseN_plan │
   └───────────┘        └─────────────┘
                               │
                               ▼
                        ┌─────────────┐
                        │ coordinator │ — reviews plan, creates task specs
                        └─────────────┘
                               │
                               ▼
                  docs/workflow/tasks/task_NNN_*.md
                  docs/workflow/task_queue.md (backlog)

/run-phase
        │
        ▼ (loop per task)
   ┌──────────┐         ┌──────────┐         ┌─────────────┐
   │ executor │ ──────▶ │  report  │ ──────▶ │ coordinator │
   └──────────┘         └──────────┘         └─────────────┘
                                                   │
                              APPROVED             │ NEEDS REVISION
                              ▼                    ▼
                  next task ←─                 same task ←─ (with appended fixes)

(after all tasks)
        │
        ▼
   ┌─────────────┐
   │ coordinator │ — phase-wide review → sign-off OR new fix tasks
   └─────────────┘
        │
        ▼
   wait for user → commit on request
```

## The three base agents

| Agent | Model | Owns | Never does |
|-------|-------|------|------------|
| `architect` | opus | `docs/workflow/planning/phaseN_plan.md` | task specs, source code, queue updates |
| `coordinator` | opus | task specs, code reviews, queue, sign-offs | implementation, running tests/builds |
| `executor` | sonnet | source code, delivery report | planning, queue updates, commits |

Models are pinned in YAML front-matter on each agent file. They apply regardless of the calling session's model.

## Hard rules baked into the templates

- **Cross-review independence.** When the coordinator reviews executor output, it receives only the task spec and the list of changed files. It reads the code directly, never the executor's narrative. This catches "the executor convinced me it's fine" failures.
- **Plan vs spec separation.** The phase plan is a high-level index — task title + one-line scope. Detailed task content (acceptance criteria, files touched, implementation notes) lives only in the per-task spec. Duplication is treated as a review failure.
- **NEEDS REVISION → append, don't fork.** When the coordinator returns NEEDS REVISION, it appends a `## Fix Requirements (Round N)` section to the existing task spec. The same task goes back to the executor. No new task file, no queue change.
- **Phase-wide review fixes are different.** Issues found in the cross-cutting phase review do produce new fix-task spec files (added to the backlog) — because they may span multiple original tasks.
- **Escalation policy.** Technical ambiguity → agent decides and notes it. Business/domain ambiguity → agent stops and asks the user.
- **Commit policy.** Coordinator commits only when explicitly asked. Always Conventional Commits. Always specific files staged — never `git add -A`.

## Where project specifics live

CAW agent and command files are project-agnostic on purpose. All project-specific context is read from `CLAUDE.md`:

- Project name, description, stack
- Quality-gate commands (what `executor` runs after each task)
- Module / architecture rules (what `architect` and `coordinator` enforce)
- Canonical domain documents (what every agent must read when their work touches the domain)
- The list of available specialized agents and the conditions to spawn them
- Workflow language (default English)
- Phase-terminology disambiguation (if "phase" is already used at another level in the project)

This is the single seam between the universal template and the specific project. If the workflow misbehaves, fix `CLAUDE.md` first; resort to overriding agent files only if the base behaviour itself is wrong for the project.

## Specialized agents

Some projects benefit from extra agents that the base three can spawn — for example a `data-analyst` for inspecting unfamiliar CSV/XLSX files before parsing, or a `swiftui-porter` for translating React components to SwiftUI.

Specialized agents are project-specific and live alongside the base three in `.claude/agents/`. They are referenced by `CLAUDE.md` (so the base agents know they exist and when to spawn them). The CAW kit ships two examples in `docs/extra-agents/` that you can copy and adapt.

See [customization.md](customization.md) for the full add-an-agent recipe.

## Artefact layout

```
docs/workflow/
├── task_queue.md                       # Current task + backlog + completed
├── planning/
│   ├── project_plan.md                 # Roadmap of all phases
│   └── phaseN_plan.md                  # One file per phase
├── tasks/
│   └── task_NNN_{slug}.md              # One file per task
└── reports/
    ├── task_NNN_report.md              # Executor delivery report
    └── task_NNN_coordinator_review.md  # Coordinator code review
```

This is the only valid layout for CAW artefacts. Both base commands enforce it strictly — if an agent writes documentation elsewhere, the orchestrator corrects it before continuing.
