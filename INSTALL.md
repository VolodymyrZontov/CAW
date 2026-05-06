# CAW Install Guide (for Claude Code)

You are installing the CAW (Coordinated Agent Workflow) into a target project. Follow these steps strictly. Do not skip steps. Do not paraphrase template files when copying them.

This document is an instruction set, not a reference. Read it top to bottom and execute it.

---

## Step 0 — Verify destination

1. Determine the **CAW source path** — where this `INSTALL.md` lives. The user may have given you a local path or a URL. Resolve it to a concrete location you can read files from. If a URL was given, fetch the file tree at that URL to know what to copy from.
2. Determine the **target project path** — the user's current working directory in their Claude Code session. Confirm with the user in one sentence:
   > "Install CAW from `<CAW_SOURCE>` into `<TARGET>`?"
3. If the target is the CAW source repo itself (the user is sitting inside the install kit, not in a real project), STOP. Tell them: "You're inside the CAW source repo. Switch to the target project and re-run." Do not proceed.

---

## Step 1 — Detect existing CAW install

Check whether any of these already exist in the target project:

- `.claude/agents/architect.md`
- `.claude/agents/coordinator.md`
- `.claude/agents/executor.md`
- `.claude/commands/new-phase.md`
- `.claude/commands/run-phase.md`
- `docs/workflow/task_queue.md`
- `docs/workflow/planning/project_plan.md`
- `CLAUDE.md`

If any exist, list them to the user and ask:

> "Existing files detected: `<list>`. Choose: **(a)** overwrite all, **(b)** skip existing (only create what's missing), **(c)** abort."

- (a) overwrite — proceed; in Step 3/4 overwrite without further prompting.
- (b) skip — proceed; in Step 3/4 only create files that don't already exist. Exception: if `CLAUDE.md` exists, never overwrite it (see Step 4).
- (c) abort — stop immediately and report.

If nothing exists, proceed.

---

## Step 2 — Gather project info

Goal: assemble the values needed to render `CLAUDE.md` in Step 4 without forcing the user to spell out things you can read from the repo. **Read first, ask second.**

### 2a — Inspect the project autonomously

Walk the target repo and collect what you can. Don't guess; if a field isn't visible from the files, mark it **unknown** and you'll ask the user about it specifically in 2c.

| Field | Where to look |
|---|---|
| Project name | `package.json` `name`, `pyproject.toml` `[project] name`, `Cargo.toml`, `go.mod`, top-level `README*`; repo directory name as last resort |
| Description | First paragraph of `README*`, manifest `description` field |
| Stack | Manifest files, lockfiles, dominant file extensions, `Dockerfile`, framework config files |
| Quality-gate commands | `Makefile` targets, `package.json` `scripts`, `pyproject.toml` `[tool.*]`, `.github/workflows/*.yml`, `CONTRIBUTING.md`, `noxfile.py`, `tox.ini` |
| Module / architecture rules | `ARCHITECTURE.md`, `docs/architecture*`, existing `CLAUDE.md`, README architecture sections, import-linter / dependency-cruiser configs |
| Canonical domain documents | Entries under `docs/` that look like specs, glossaries, or domain references; links from README into `docs/` |
| Phase terminology | `grep -ri "phase" docs/ README*` — does the repo already use the word for a release/lifecycle concept distinct from CAW phases? |

### 2b — Pick the conversation language

For all user-facing prompts in this step (and the rest of the install) use **the language the user has been speaking to you in this session**. The English text in this document is the instruction set for you; what you say to the user must be in their language.

For `WORKFLOW_LANGUAGE` (the language CAW workflow markdown will be written in — task specs, reports, plans) default to **English**, but if the project's existing docs/README are clearly in another language, propose that language as the default and let the user override.

### 2c — Confirm with the user in one consolidated message

Send a single message — in the user's language — listing every field with what you found, and explicitly marking items you didn't find. Ask the user to confirm or correct. Shape (render in the user's language; keep field names in English so they map 1:1 to Step 4 placeholders):

```
I scanned the repo. Please confirm or correct:

- Project name: <value>
- Description: <value>
- Stack: <value>
- Quality-gate commands: <value | "not found">
- Module / architecture rules: <value | "not found">
- Canonical domain documents: <list | "not found">
- Workflow language: <value>
- Phase terminology: <"no conflict" | "already used as: ...">

For anything I missed, fill it in or reply with "TBD" / "none".
```

Wait for the full response. Apply corrections. Do not proceed to Step 3 until every field has a concrete value — `TBD` and `none` are acceptable terminal values per Step 4 placeholder rules.

If any answer is unclear or self-contradictory, ask one targeted clarifying question before moving on. Do not fabricate. Wrong values here lock bad context into every future agent invocation.

---

## Step 3 — Copy verbatim files

Copy these files **byte-for-byte** from `<CAW_SOURCE>` into the target project. Do not edit, summarise, or "improve" them. They are designed to be project-agnostic; all project specifics belong in `CLAUDE.md`.

| Source | Destination |
|--------|-------------|
| `templates/.claude/agents/architect.md` | `.claude/agents/architect.md` |
| `templates/.claude/agents/coordinator.md` | `.claude/agents/coordinator.md` |
| `templates/.claude/agents/executor.md` | `.claude/agents/executor.md` |
| `templates/.claude/commands/new-phase.md` | `.claude/commands/new-phase.md` |
| `templates/.claude/commands/run-phase.md` | `.claude/commands/run-phase.md` |
| `templates/.claude/settings.json` | `.claude/settings.json` *(only if `.claude/settings.json` doesn't already exist — never overwrite an existing one)* |

Then create empty directories if they don't already exist:

- `docs/workflow/planning/`
- `docs/workflow/tasks/`
- `docs/workflow/reports/`

Then copy these workflow skeletons (only if the destination doesn't already exist):

| Source | Destination |
|--------|-------------|
| `templates/docs/workflow/task_queue.md` | `docs/workflow/task_queue.md` |
| `templates/docs/workflow/planning/project_plan.md` | `docs/workflow/planning/project_plan.md` |

---

## Step 4 — Generate CLAUDE.md

Read `<CAW_SOURCE>/templates/CLAUDE.md.template`. Substitute the placeholders with answers from Step 2:

| Placeholder | Source (field from Step 2) |
|-------------|--------|
| `{{PROJECT_NAME}}` | Project name |
| `{{PROJECT_DESCRIPTION}}` | Description |
| `{{STACK}}` | Stack |
| `{{QUALITY_GATES}}` | Quality-gate commands — render as a fenced bash block; if "TBD", insert a single line: `# TBD — fill in once quality gates are defined.` |
| `{{MODULE_RULES}}` | Module / architecture rules — free-form prose; if "none", write `_None — this project has no enforced module layering._` |
| `{{DOMAIN_DOCS}}` | Canonical domain documents — render as a markdown table of `Path | Purpose`; if "none", write `_None — this project has no canonical domain documents yet._` |
| `{{EXTRA_AGENTS_ROWS}}` | Always emit an empty string. Extra agents are out of scope for the installer; the user can add them later by dropping files into `.claude/agents/` and a row into the agent table. |
| `{{WORKFLOW_LANGUAGE}}` | Workflow language |
| `{{PHASE_TERMINOLOGY}}` | Phase terminology — if "no conflict", write `_None — "phase" in this repo always means CAW phase._`; if a conflict exists, write a short paragraph naming the macro-phase term, where it's documented, and the rule that "phase" in CAW conversations means CAW phase by default. |

**If `CLAUDE.md` already exists in the target project:**

- Do NOT overwrite it.
- Render the substituted template into a temp string.
- Show the user the rendered template and ask: "Existing `CLAUDE.md` found. I will not overwrite it. Here is the CAW-flavoured template — review and tell me which sections to merge in." Wait for the user's decision before editing the existing `CLAUDE.md`.

If `CLAUDE.md` does not exist, write the rendered template to `CLAUDE.md`.

---

## Step 5 — Final report

Tell the user:

```
CAW installed into <TARGET>.

Created:
  .claude/agents/architect.md
  .claude/agents/coordinator.md
  .claude/agents/executor.md
  .claude/commands/new-phase.md
  .claude/commands/run-phase.md
  .claude/settings.json (or "skipped — already existed")
  CLAUDE.md (or "skipped — review template against your existing CLAUDE.md")
  docs/workflow/task_queue.md
  docs/workflow/planning/project_plan.md
  docs/workflow/tasks/ (empty)
  docs/workflow/reports/ (empty)

Next steps:
  1. Review CLAUDE.md and adjust quality-gate commands, module rules, and domain docs as needed.
  2. (Optional) Add a high-level roadmap to docs/workflow/planning/project_plan.md.
  3. Run /new-phase <one-line description> to plan the first phase.
  4. Run /run-phase once the queue has tasks.
```

---

## Rules for the installer (you, Claude)

- **Speak the user's language.** Every prompt, confirmation, and report you send to the user during this install must be in the language they are using to talk to you. The English text in this document is for you, not for them.
- Do **not** edit the agent or command templates while copying. They are designed to read all project-specific context from `CLAUDE.md`. Any deviation is drift.
- Do **not** add commentary, headers, or extra notes inside copied files.
- Do **not** invent answers to the Step 2 fields. If the repo doesn't reveal an answer and the user is unsure, accept "TBD" or "none" and document that — don't fabricate.
- Do **not** overwrite an existing `CLAUDE.md` or `.claude/settings.json` without explicit user instruction.
- Do **not** skip the user-confirmation steps (Step 0, Step 1 conflict prompt, Step 4 existing-CLAUDE.md prompt).
- If you encounter anything ambiguous in the source kit (a missing template file, an unexpected layout), stop and tell the user — don't improvise.
