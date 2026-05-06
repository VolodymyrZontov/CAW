You are the orchestrator. Your job is to run the full phase planning workflow automatically.

Phase description from the user: $ARGUMENTS

Follow these steps in order. Do not skip steps. Do not ask the user questions unless instructed below.

---

## Step 1 — Architect: write the phase plan

Spawn an Agent with `subagent_type: architect` and this prompt (include it verbatim, replacing {DESCRIPTION} with the user's phase description):

> You are the architect for this project. Your role and rules are in `.claude/agents/architect.md` — read it first. Project-specific context (stack, module rules, canonical domain docs, specialized agents) is in `CLAUDE.md` — read that next.
>
> Design a new implementation phase for: {DESCRIPTION}
>
> Follow your role instructions exactly. Write the plan file when done.

Wait for the agent to finish. Note the plan file path it created (e.g. `docs/workflow/planning/phaseN_plan.md`).

---

## Step 2 — Check for Open Questions

Read the plan file. Find the `## Open Questions` section.

If it contains anything other than `_None_`:

- Present the open questions to the user.
- Wait for answers.
- Spawn the architect agent again (`subagent_type: architect`) with: "Update the plan at {plan_path} — here are the answers to the open questions: {answers}. Revise the plan accordingly."
- Re-read the plan after revision.

If Open Questions is `_None_` or empty: proceed immediately.

---

## Step 3 — Coordinator: review plan and create task specs

Spawn an Agent with `subagent_type: coordinator` and this prompt (replace {PLAN_PATH} with the actual path):

> You are the coordinator for this project. Your role and rules are in `.claude/agents/coordinator.md` — read it first. Project-specific context is in `CLAUDE.md` — read that next.
>
> Review the phase plan at {PLAN_PATH}.
>
> 1. Review the plan for technical correctness and completeness. Resolve all technical issues yourself. If you find business-logic gaps not covered by the plan's Open Questions, list them in your review but do NOT block — note them as "Coordinator raised: ..." in the review section.
> 2. If the plan is APPROVED (or APPROVED WITH NOTES): create all task specs for this phase.
> 3. Update `docs/workflow/task_queue.md` — add new tasks to the backlog. Do not change the Current Task.
>
> Follow your role instructions exactly.

Wait for the agent to finish.

---

## Step 4 — Disagreement or coordinator questions

Read the coordinator review appended to the plan file.

If verdict is NEEDS REVISION:

- Spawn architect (`subagent_type: architect`): "The coordinator reviewed your plan at {PLAN_PATH} and requested revisions. Read the Coordinator Review section at the bottom of the plan. Address each issue and update the plan."
- Then re-run Step 3 (one more round only).
- If still NEEDS REVISION after round 2: report to user — "Architect and coordinator could not agree. Here are the open issues: {issues}. Please decide."

If coordinator raised business-logic questions ("Coordinator raised: ..."):

- Present them to the user and wait for answers before considering the workflow complete.

---

## Step 5 — Report to user

Read `docs/workflow/task_queue.md` and report:

```
Phase N plan created: docs/workflow/planning/phaseN_plan.md
Tasks created: NNN, NNN, NNN
Added to backlog: X tasks
Current task unchanged: Task NNN (or "no current task")

Ready to execute: /run-phase
```
