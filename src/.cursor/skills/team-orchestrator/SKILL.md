---
name: team-orchestrator
description: Team pipeline contract covering run folders, full/fast modes, per-stage file I/O, and 40-plan.plan.md format. Skill id is team-orchestrator (folder name); the doc covers every stage (analyst … code-reviewer), not only the orchestrator agent. Use when coordinating runs or when any team-pipeline-* agent needs the full spec.
---

# Team pipeline contract

The Cursor skill id and folder are **`team-orchestrator`**, but this file is the **single contract for the whole multi-stage pipeline**: what each stage reads and writes, `full` vs `fast`, and the `40-plan.plan.md` format. The agent **`team-pipeline-orchestrator`** is only one consumer; **`team-pipeline-analyst`**, **`team-pipeline-developer`**, and the other stage agents should use the same document for gates and paths.

## When to use

- Starting or continuing a **versioned run** under `.cursor/tasks/runs/`.
- Invoking **team-pipeline-*** agents or `Task` subagents for a single stage.
- Resolving **full** vs **fast** mode and which files must exist before each stage.

Paths below are **relative to the active run directory** unless noted.

## Workspace root (critical)

All paths that start with **`.cursor/`** (e.g. `.cursor/tasks/runs/LATEST`, `.cursor/tasks/templates/00-brief.md`, `.cursor/skills/team-orchestrator/SKILL.md`) are **relative to the Cursor workspace root** — the **folder opened as the project** in this chat — **not** relative to:

- the user’s **global** Cursor config (`~/.cursor`, `%USERPROFILE%\.cursor`, `C:\Users\…\.cursor`), where **agent `.md` files** may live;
- the on-disk path to a copied agent definition.

**Implications:**

- If `.cursor/tasks/` does **not** exist under the **workspace root**, this project has **no** in-repo pipeline tree: **bootstrap** must **create** `<workspace>/.cursor/tasks/runs/` (and siblings as needed), **or** the user must copy the packaged `.cursor/` (tasks, skills, rules) into the opened project. Do **not** expect `tasks/` to appear next to global `agents/`.
- **`run=`** may be an **absolute** path (Windows `C:\…`, UNC, WSL `/mnt/…`) so the run folder is unambiguous when the workspace is multi-root or unusual.
- If the skill file is not present under the workspace `.cursor/skills/`, still follow this document’s rules (agents may load from global skills); **artifact I/O** always targets the **workspace** unless the user supplies an absolute `run=`.

## Run directory

Resolve the **active run folder** (the directory that contains `00-brief.md`, `10-analyst.md`, …).

### Parsing `run=` (step 1 — always before `LATEST`)

- **Where to look:** the **entire** user-visible message for this turn — including text **after** a slash command (e.g. `/team-pipeline-critical-reviewer …`) or agent name. Cursor may prepend the command; the rest of the line/body still counts.
- **Pattern:** case-insensitive `run`, optional whitespace, `=`, optional whitespace, then the path. Examples: `run=.cursor/tasks/runs/foo_01`, `run = .cursor\tasks\runs\foo_01`, `Проверь run=.cursor/tasks/runs/foo_01`.
- **Windows backslashes:** treat `\` as a path separator — **normalize** to `/` for repo-relative paths (e.g. `.cursor\tasks\runs\20260325_o2k-exchange-audit_01` → `.cursor/tasks/runs/20260325_o2k-exchange-audit_01`) before resolving from the workspace root.
- **Quoting:** if the path is wrapped in `"..."`, strip quotes and trim.
- **Absolute paths:** `C:\...`, `D:/...`, UNC — use as-is (after normalizing separators if your tools require it).
- **If multiple `run=` appear:** use the **last** non-empty one (closest to user intent in long prompts).
- **If `run=` is present but the path is empty or invalid:** stop and ask the user to fix it — **do not** fall through to `LATEST` silently (that hides mistakes).

**Anti-pattern:** Reading `LATEST` when the user already wrote `run=…` in the same message is a **contract violation**.

**Resolution order:**

1. If **§ Parsing `run=`** yields a path → use it **only** — **do not** read `.cursor/tasks/runs/LATEST` for resolution in this invocation. **No affinity check** when the user passed `run=`; you may update artifacts in that folder.
2. **Else** read `.cursor/tasks/runs/LATEST` — **first line** is the **candidate** run path (absolute or repo-relative). Skip leading empty lines if needed. Then apply **§ Same run vs new run** below before treating that folder as final.
3. If still unknown:
   - **Bootstrap** (next subsection) **if** you are **`team-pipeline-orchestrator`** or **`team-pipeline-analyst`** and the **Bootstrap conditions** are satisfied — then the new folder becomes the active run.
   - **Else** **stop** and tell the user exactly what to do (do not guess a path):
     - **Create a run** per `.cursor/tasks/README.md` (folder `.cursor/tasks/runs/<YYYYMMDD>_<slug>_<seq>/`, copy templates, fill `00-brief.md`).
     - Write **one line** — the path to that run folder — into `.cursor/tasks/runs/LATEST`.
     - **Or** re-invoke the stage with `run=<path-to-run-folder>` in the message.
   - **Other stage agents** (architect, planner, …): if the run is still unknown after steps 1–2, **always** stop with those recovery steps — **do not** bootstrap a new run.

**Not a run path:** `.cursor/plans/*.md` (and similar Cursor **Plan** UI exports) live **outside** `tasks/runs/`. Do **not** use their folder or filename to infer `LATEST` or substitute for `run=`. You may still **read** such a file as user-provided context once a run directory is resolved.

### Same run vs new run (do not clobber another audit)

`LATEST` remembers the **last** run; a **new chat** or **new task** must **not** automatically reuse it if the **subject** differs.

**When this applies:** the run path came **only** from step 2 (`LATEST`), **not** from `run=`. If the message includes `new_run=1` or `new_run` or explicit “новый прогон / new pipeline run / separate run”, treat the request as **new** (skip matching; create a new folder).

**Check (after resolving a folder from `LATEST`):**

1. Open that folder’s **`00-brief.md`** (and optionally **`manifest.json`**: `topic_slug`, `run_id`, `notes`).
2. Compare **problem scope** in the brief with **this message**: product/module/audit target, client, feature area, and stated goal.
3. **Same initiative** — safe to reuse: user clearly **continues** the same work (refine analysis, “обнови `10-analyst`”, same module and goal as the brief), or the brief and request **obviously** describe the same engagement.
4. **Different initiative** — **do not** write stage artifacts into that folder. **Analyst** or **orchestrator** must **create a new** run directory (new `short-topic-slug` and/or next `seq` under `.cursor/tasks/runs/`), fill a **fresh** `00-brief.md` from **this** chat, set **`LATEST`** to the **new** path, then continue. **Leave the old run folder unchanged** (historical record).
5. **Ambiguous:** prefer **new run** over overwriting an unrelated audit (avoids data loss). Optionally ask one short confirmation; if no reply assumed, default to **new run**.

**Other stages** (architect, planner, …) when using `LATEST`: if after reading `00-brief.md` the task **clearly** does not match the brief, **stop** — do not write; tell the user to pass `run=<correct-folder>` or run **analyst** / **orchestrator** to create a new run and fix `LATEST`.

**Explicit `run=`** overrides this subsection for path choice (user responsibility).

### Bootstrap (create run + `LATEST`)

**Who:** only **`team-pipeline-orchestrator`** or **`team-pipeline-analyst`**.

**When (any of):**
- The message includes a machine flag: `bootstrap_run`, `ensure_run=1`, `create_run=1`, or **`new_run` / `new_run=1`** (always a **new** folder; update `LATEST`).
- **§ Same run vs new run** requires a **new** folder (mismatch or ambiguous vs `LATEST` run’s brief).
- The user clearly asks to **start a new pipeline run**, **create a run folder**, or equivalent.
- The user invoked this analyst/orchestrator with **enough substance** to fill `00-brief.md` (problem statement, goal, constraints) and intent is clearly to produce pipeline artifacts — not a one-line trivia question.

**When not:** intent is ambiguous, the user forbids creating files, or you would have to invent a topic with no basis in the message — then **stop** with manual recovery steps only.

**Procedure** (atomic; then continue the same invocation with the new path):
1. Ensure `.cursor/tasks/runs/` exists (create the directory if missing).
2. Choose `YYYYMMDD` (today, UTC or local — be consistent in one run), `short-topic-slug` (lowercase, hyphens, from the task title), and `seq` (`01`, or next free by listing `runs/` — see `.cursor/tasks/README.md`).
3. Create `.cursor/tasks/runs/<YYYYMMDD>_<slug>_<seq>/`.
4. Copy `.cursor/tasks/templates/00-brief.md` into the new folder; **edit** it with the user’s problem/context (and optional pointers to `.cursor/plans/…` or other files — by path, not as run root).
5. Copy `.cursor/tasks/templates/manifest.example.json` → `manifest.json` in that folder (rename); set `"mode": "full"` or `"fast"` if the user specified it, else default **fast** unless the conversation already chose full.
6. Write `.cursor/tasks/runs/LATEST` — **exactly one non-empty line**: repo-relative path to the new folder (same path you will use as `run=`).
7. In the reply, **briefly state** what was created (folder path + that `LATEST` was set). Do **not** delete or overwrite sibling run folders.

**Allowed files/dirs to create during bootstrap:** the new run directory, `00-brief.md`, `manifest.json` inside it, and `.cursor/tasks/runs/LATEST` (and `runs/.gitkeep` is optional). Do not create spurious files outside this layout.

## Modes

| Mode | Stages |
| ---- | ------ |
| **full** | analyst → architect → critical-reviewer → planner → plan_reviewer → developer → code_reviewer |
| **fast** | analyst → planner → developer (optional code_reviewer) |

- Set `mode` in `manifest.json` and/or pass `mode=full` / `mode=fast` in chat when ambiguous.
- **fast:** no `20-architecture.md` required before planner; planner **must** document assumptions and unknowns in the plan body.

## Artifact table (read / write)

| Stage | Read | Write |
| ----- | ---- | ----- |
| analyst | `00-brief.md` | `10-analyst.md`, `manifest.json` |
| architect | `10-analyst.md` | `20-architecture.md`, `manifest.json` |
| critical-reviewer | `10-analyst.md`, `20-architecture.md` | `30-critical-review.md`, `manifest.json` |
| planner (full) | `10-analyst.md`, `20-architecture.md`, optional `30-critical-review.md` | `.cursor/plans/<run_id>/40-plan.plan.md` + run link file `40-plan.plan.path`, `manifest.json` |
| planner (fast) | `10-analyst.md` | `.cursor/plans/<run_id>/40-plan.plan.md` + run link file `40-plan.plan.path`, `manifest.json` |
| plan_reviewer | `.cursor/plans/<run_id>/40-plan.plan.md` (via `40-plan.plan.path`) | `50-plan-review.md`, `manifest.json` |
| developer (full) | `.cursor/plans/<run_id>/40-plan.plan.md` (via `40-plan.plan.path`), `50-plan-review.md` | code, `60-implementation-notes.md`, update plan `todos[].status`, `manifest.json` |
| developer (fast) | `.cursor/plans/<run_id>/40-plan.plan.md` (via `40-plan.plan.path`) | code, `60-implementation-notes.md`, update plan `todos[].status`, `manifest.json` |
| code_reviewer | `60-implementation-notes.md` (+ repo as needed) | `70-code-review.md`, `manifest.json` |

Before each stage: verify **required inputs** exist; if not, stop and name the missing producer stage.

Optional **Task** subagents from project rules may extend the chain; keep new artifacts in the **same run folder**.

## Planner artifact: `.cursor/plans/<run_id>/40-plan.plan.md` (canonical)

The canonical plan lives under:

- `.cursor/plans/<run_id>/40-plan.plan.md`

Where:

- `<run_id>` = the **run folder name**: `<YYYYMMDD>_<short-topic-slug>_<seq>` (basename of the resolved `run=` directory).

Inside the run directory, the planner must also create/update a **link file**:

- `40-plan.plan.path` — **one line**: repo-relative (or absolute) path to the canonical plan file above.

This makes the run folder self-contained for stage gating without duplicating the plan content in two places.

### File shape

1. **YAML frontmatter** (between `---` lines), fields:
   - `name` — short plan title (plain string, comparable to Cursor Plan name).
   - `overview` — 1–2 sentences (string).
   - `todos` — list of objects: `id` (stable machine id), `content` (human text), `status` (e.g. `pending`; update as work progresses if the team agrees).
   - `isProject` — boolean; use `false` for a normal run artifact unless project rules say otherwise.

2. **Markdown body** after the closing `---`:
   - Leading `#` heading (may expand on `name`).
   - Detailed sections: milestones, ordered work, dependencies, risks — whatever developers need.
   - `## Assumptions and unknowns` — **required in fast mode** by meaning; in full may be brief or point to architecture / critical review.
   - `## Test / verification checklist` — concrete checks before calling work “done”.

Do **not** duplicate the todo list as a second checklist in the body: the canonical checklist for tooling is **`todos` in YAML**. In the body, reference the same `id` values when describing work.

### Task format in the body (for developers)

Each concrete task should be scannable, for example:

```markdown
#### Task [id]: [Title]
- **Description:** What to do
- **Inputs:** Pointers to `10-analyst.md` / `20-architecture.md` / code areas
- **Acceptance criteria:** How to verify
- **Dependencies:** Prior task ids or milestones
```

Use **`todos[].id` in frontmatter** aligned with these task ids (or a clear mapping).

### Plan mode (Cursor UI)

Only the **user** can turn on Chat **Plan mode**. The pipeline’s source of truth is still the canonical plan file **`.cursor/plans/<run_id>/40-plan.plan.md`** (whether or not a draft was first agreed in Plan mode).

## Artifact completeness (all pipeline markdown outputs)

Every stage that writes a **markdown file** in the run folder (`10-analyst.md` … `70-code-review.md`) must treat that file as the **canonical** record for downstream stages and humans.

- **Same depth as chat:** Put the **full** narrative you would give in the chat (sections, tables, numbered findings, rationale) **into the file** for this invocation. Do **not** leave the file as a stub or short outline while the chat holds the long version.
- **After writing:** The chat reply may be a short pointer or executive summary; the file must already be complete.
- **Exception:** Only if the user **explicitly** asks for a deliberately brief artifact.
- **Planner (`.cursor/plans/<run_id>/40-plan.plan.md`):** The **YAML frontmatter plus the full markdown body** (milestones, task detail, assumptions, verification checklist) must reflect the complete plan — not frontmatter-only or an empty/minimal body while detail stays in chat.
- **Developer:** **`60-implementation-notes.md`** must include the same level of detail as the chat (files touched, key decisions, commands run, test results, caveats) — not a one-line “done” note.

## Quality bar (per stage, concise)

- **Analyst (`10-analyst.md`):** Use cases / user-visible behavior, acceptance criteria, open questions; no architecture decisions or implementation. Must satisfy **Artifact completeness** above.
- **Architect (`20-architecture.md`):** Boundaries, interfaces, data flow, risks, alignment with existing codebase (read-only on product code for design). Must satisfy **Artifact completeness** above.
- **Critical reviewer (`30-critical-review.md`):** Blockers, inconsistencies between analyst + architecture, go/no-go signals. Must satisfy **Artifact completeness** above.
- **Planner (`40-plan.plan.md`):** Full plan in file per SKILL **Planner artifact** + **Artifact completeness** above.
- **Plan reviewer (`50-plan-review.md`):** Sequencing, missing deps, scope creep vs `40-plan.plan.md`; actionable edits. Must satisfy **Artifact completeness** above.
- **Developer:** Implement against `40-plan.plan.md` (+ `50-plan-review.md` in full); minimal tests on critical paths; **`60-implementation-notes.md`** per **Artifact completeness** above.
- **Code reviewer (`70-code-review.md`):** Bugs, security, regressions, test gaps vs stated criteria. Must satisfy **Artifact completeness** above.

## `manifest.json`

Each stage sets its `stages.<role>` to `done` and preserves other keys. Bootstrap from `.cursor/tasks/templates/manifest.example.json` if missing.

## Orchestration hints

- One **role per invocation**; pass `run=` and `mode=` when relevant.
- After creating or switching runs, set `.cursor/tasks/runs/LATEST` to the run path (single line).

## Safety: `.cursor/plans/<run_id>/` is append/update-only

For plan directories under `.cursor/plans/<run_id>/`:

- **Do not delete** any files in `.cursor/plans/<run_id>/`.
- **Do not fully clear** a plan file (e.g. replacing the entire file with empty content).
- Allowed edits:
  - **Planner** may create and fully update `.cursor/plans/<run_id>/40-plan.plan.md`.
  - **Developer** may update **only** `todos[].status` (and other minimal bookkeeping agreed in the plan) while implementing.
  - Other stages treat `.cursor/plans/<run_id>/` as **read-only**.

Only override these safety rules if the **user explicitly asks** to delete/clear, or the stage’s own contract requires rewriting its canonical artifact.

## Safety: `.cursor/tasks/runs/<run_id>/` is historical record (no deletes)

For run directories under `.cursor/tasks/runs/<run_id>/`:

- **Do not delete** files or folders inside a run directory (including the run directory itself).
- **Do not fully clear** stage artifacts (e.g. replacing an artifact with empty content) as a shortcut.
- Allowed updates (overwrite-in-place) are limited to **the current stage’s own outputs** as defined in the artifact table for this contract:
  - analyst → `10-analyst.md`, `manifest.json`
  - architect → `20-architecture.md`, `manifest.json`
  - critical-reviewer → `30-critical-review.md`, `manifest.json`
  - planner → `40-plan.plan.path`, `manifest.json` (canonical plan lives in `.cursor/plans/...`)
  - plan_reviewer → `50-plan-review.md`, `manifest.json`
  - developer → `60-implementation-notes.md`, `manifest.json`
  - code_reviewer → `70-code-review.md`, `manifest.json`
- If a stage needs to “redo” work for a **new topic**, do **not** mutate the old run: create a **new** run folder (see § Same run vs new run / bootstrap).

Only override these safety rules if the **user explicitly asks** to delete/clear, and the request is scoped to the exact paths to be removed.
