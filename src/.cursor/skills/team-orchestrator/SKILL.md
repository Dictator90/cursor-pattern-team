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

## Run directory

1. If the user message contains `run=<path>`, use that path (absolute or repo-relative).
2. Else read `.cursor/tasks/runs/LATEST` — **first line** is the active run path.
3. If still unknown: stop; user must create a run per `.cursor/tasks/README.md` or pass `run=`.

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
| planner (full) | `10-analyst.md`, `20-architecture.md`, optional `30-critical-review.md` | `40-plan.plan.md`, `manifest.json` |
| planner (fast) | `10-analyst.md` | `40-plan.plan.md`, `manifest.json` |
| plan_reviewer | `40-plan.plan.md` | `50-plan-review.md`, `manifest.json` |
| developer (full) | `40-plan.plan.md`, `50-plan-review.md` | code, `60-implementation-notes.md`, `manifest.json` |
| developer (fast) | `40-plan.plan.md` | code, `60-implementation-notes.md`, `manifest.json` |
| code_reviewer | `60-implementation-notes.md` (+ repo as needed) | `70-code-review.md`, `manifest.json` |

Before each stage: verify **required inputs** exist; if not, stop and name the missing producer stage.

Optional **Task** subagents from project rules may extend the chain; keep new artifacts in the **same run folder**.

## Planner artifact: `40-plan.plan.md`

Use the **`.plan.md` suffix** so the file matches Cursor Plan-style artifacts (same idea as plans under `.cursor/plans/`).

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

Only the **user** can turn on Chat **Plan mode**. The pipeline’s source of truth is still **`40-plan.plan.md`** in the run directory, filled to this spec (whether or not a draft was first agreed in Plan mode).

## Quality bar (per stage, concise)

- **Analyst (`10-analyst.md`):** Use cases / user-visible behavior, acceptance criteria, open questions; no architecture decisions or implementation.
- **Architect (`20-architecture.md`):** Boundaries, interfaces, data flow, risks, alignment with existing codebase (read-only on product code for design).
- **Critical reviewer (`30-critical-review.md`):** Blockers, inconsistencies between analyst + architecture, go/no-go signals.
- **Plan reviewer (`50-plan-review.md`):** Sequencing, missing deps, scope creep vs `40-plan.plan.md`; actionable edits.
- **Developer:** Implement against `40-plan.plan.md` (+ `50-plan-review.md` in full); minimal tests on critical paths; record commands/tests in `60-implementation-notes.md`.
- **Code reviewer (`70-code-review.md`):** Bugs, security, regressions, test gaps vs stated criteria.

## `manifest.json`

Each stage sets its `stages.<role>` to `done` and preserves other keys. Bootstrap from `.cursor/tasks/templates/manifest.example.json` if missing.

## Orchestration hints

- One **role per invocation**; pass `run=` and `mode=` when relevant.
- After creating or switching runs, set `.cursor/tasks/runs/LATEST` to the run path (single line).
