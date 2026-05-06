# Customising CAW per project

CAW is intentionally minimal at install time: three universal agents, two universal commands, one `CLAUDE.md` describing the project. All per-project tuning happens through `CLAUDE.md` first; only override agent files when the base behaviour itself is wrong for your project.

## Where to put what

| You want to … | Do this |
|---------------|---------|
| Tell agents about your stack, build commands, architecture rules | Edit `CLAUDE.md` (sections: Stack, Commands, Module / architecture rules). |
| Point agents at canonical domain docs they must read | Edit `CLAUDE.md` § Canonical domain documents. Add the path and a one-line purpose. |
| Add a specialized agent (data-analyst, UI-port helper, security reviewer, …) | Create `.claude/agents/<name>.md` and add a row to the agent table in `CLAUDE.md`. See "Adding a specialized agent" below. |
| Change the workflow language | Edit `CLAUDE.md` § Agent workflows § Language rule. |
| Disambiguate "phase" if your project already uses the word at another level | Edit `CLAUDE.md` § Phase terminology. |
| Change quality-gate commands | Edit `CLAUDE.md` § Commands. The executor reads this directly. |
| Change cross-review behaviour, plan/spec separation, escalation policy | Edit the agent file in `.claude/agents/` — but only if the base behaviour is genuinely wrong for your project. Diverging from the template is drift. |

## Adding a specialized agent

The kit ships two examples in `docs/extra-agents/`:

- `data-analyst.example.md` — read-only inspection of CSV/XLSX/Parquet sources before parsing.
- `swiftui-porter.example.md` — port React/Tailwind components to SwiftUI.

Recipe:

1. Pick the closest example (or none if your agent is unique).
2. Copy it to `.claude/agents/<name>.md` and adjust:
   - `name:` and `description:` in YAML front-matter.
   - The "Role" paragraph.
   - The "When to spawn" section — describe the exact triggering condition. Be specific. "When the architect plans a task that touches a CSV file the project hasn't seen before" beats "for data tasks".
   - The "What it reads" / "What it does not do" sections — keep tight, single-purpose.
   - The "Output format" section — define exactly what the spawning agent receives back.
3. Add a row to the agent table in `CLAUDE.md`:
   ```
   | `<name>` | <model> | <one-line role>. Spawned by <architect|coordinator|executor> when <condition>. |
   ```
4. Verify the base agents will actually invoke it. The base templates include a generic "if `CLAUDE.md` lists specialized agents with a 'spawn when …' condition that matches, spawn that agent" rule — so as long as the table row is clear and unambiguous, the base agents will pick it up.

Keep specialized agents narrow. One agent = one read-only or one transformation responsibility. If you're tempted to give a specialized agent multiple unrelated jobs, split it.

## Overriding base agent behaviour

If you find yourself wanting to change `architect.md`, `coordinator.md`, or `executor.md` itself, ask first: can this be expressed as a `CLAUDE.md` rule instead? If yes, prefer that — it keeps the project alignable with future CAW updates.

If the change really has to live in the agent file:

1. Edit `.claude/agents/<agent>.md` in the project directly.
2. Note the change in `CLAUDE.md` under a "## CAW overrides" section so future readers (and you) know the agent is non-standard.
3. Don't bring the change back into the CAW kit unless it generalises to all projects.

## Updating CAW in an existing project

When the CAW kit changes (new rules, fixed bugs in the base agents), pulling updates into a project that already has CAW installed is a manual diff:

1. Pull the new CAW kit.
2. Diff `templates/.claude/agents/*.md` and `templates/.claude/commands/*.md` against the project's `.claude/`.
3. Apply the diffs that don't conflict with intentional project overrides (which are documented in `CLAUDE.md § CAW overrides`, per the rule above).
4. Re-render `CLAUDE.md.template` mentally — if the kit added new sections (e.g. a new top-level rule), add them to the project's `CLAUDE.md` too.

The kit deliberately doesn't ship a "CAW upgrade" command. Updating an active project's agents is judgement work; an automated overwrite would silently destroy local overrides.

## What CAW won't do for you

- **It won't decide for you what the project is.** `CLAUDE.md` only works if it's accurate. "TBD" answers during install lead to TBD agent behaviour.
- **It won't enforce your code style or test setup.** The `executor` runs whatever `CLAUDE.md § Commands` says to run. If those commands are wrong, the gate is wrong. Set them up properly.
- **It won't replace product judgement.** The escalation rule is "business/domain ambiguity → ask the user". Agents will surface questions; they won't invent answers.
