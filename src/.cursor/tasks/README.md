# Team task artifacts (`.cursor/tasks`)

Подробное руководство пользователя (RU): [README.md в корне репозитория](../../../README.md).

This directory stores **versioned run folders** for the team pipeline. Nothing here overwrites a previous run: each run gets a new directory.

## Directory layout

- `templates/` — static templates copied or referenced when starting a run (do not overwrite these with automation). Index: [templates/README.md](templates/README.md).
- `runs/` — all pipeline runs.
- `runs/<YYYYMMDD>_<short-topic-slug>_<seq>/` — one run (e.g. `20260324_catalog-filters_01`).
  - `<seq>` is `01`, `02`, … for the same date + slug. Pick the next free sequence by listing existing folders.
- `runs/LATEST` — optional pointer file: **single line**, absolute or repo-relative path to the **current** run directory. Update after creating or selecting a run; older run folders stay on disk.

## Artifact files (inside each run directory)

| File | Produced by | Purpose |
| ---- | ----------- | -------- |
| `00-brief.md` | User / orchestrator | Problem statement, constraints, links |
| `10-analyst.md` | analyst | Requirements, cases, acceptance criteria |
| `20-architecture.md` | architect | Design, boundaries, risks (full mode) |
| `30-critical-review.md` | critical-reviewer | Blockers, questions, go/no-go (full mode) |
| `40-plan.plan.md` | planner | Cursor Plan-style file: YAML frontmatter + body; task breakdown, dependencies |
| `50-plan-review.md` | plan_reviewer | Plan review notes (full mode) |
| `60-implementation-notes.md` | developer | What changed, tests/commands run |
| `70-code-review.md` | code_reviewer | Review findings |
| `manifest.json` | each stage updates | Run metadata and stage status |

See `.cursor/skills/team-orchestrator/SKILL.md` for strict read/write contracts and `full` vs `fast` mode.

## Starting a new run

1. Choose `YYYYMMDD`, `short-topic-slug` (lowercase, hyphens).
2. List `runs/` for matching `YYYYMMDD_<slug>_*` and set `<seq>` to next two-digit suffix.
3. Create the folder; copy needed files from `templates/` (see [templates/README.md](templates/README.md)) — at minimum `00-brief.md` and `manifest.json`; add others as stages require.
4. Write the path to `runs/LATEST` (one line).

## Manual stage workflow

Resolve the run directory from `runs/LATEST` or an explicit `run=` path, then open only the **input** files defined for that stage in the team-orchestrator skill.

For **explicit Cursor agents** (one per stage), use `.cursor/agents/team-pipeline-*.md` — each file documents run resolution, gates, inputs, outputs, and `manifest.json` updates. The **pipeline contract** (all stages) remains `.cursor/skills/team-orchestrator/SKILL.md` (skill id `team-orchestrator` is the folder name; the doc is not orchestrator-only).

To use the same agent definitions globally (e.g. `C:\Users\<you>\.cursor\agents`), copy `team-pipeline-*.md` from this repo; the repository copy is the source of truth for the project.
