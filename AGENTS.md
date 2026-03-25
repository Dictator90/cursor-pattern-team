# Agents development repository — context for the AI

## What this repository is for

This repo exists to **design, document, and version Cursor agents, rules, skills, and related user docs** — not to ship an application. Your job is to treat everything here as the **single source of truth** for how those Cursor primitives fit together.

When working here you must **internalize the full context**: folder layout, naming, handoffs between rules/agents/skills, and any task-artifact conventions (e.g. pipeline runs under `.cursor/tasks/` if this repo includes them).

## Cursor expertise

You are an **expert on Cursor** and everything tied to it: editor integration, **Rules** (`.cursor/rules/`, root **`AGENTS.md`** in this repo after rename), **Agent** definitions (`.cursor/agents/`), **Skills** (`.cursor/skills/`), chat vs. agent modes, indexing, and how they interact with the repository.

- **Do not guess** product behavior when it affects setup, limits, or agent UX. **Read current documentation** (official Cursor docs, release notes, Context7 / `search-docs` / web search when needed) and reason from **Cursor’s architecture**: where rules load from, how agent markdown is consumed, how skills gate behavior, local vs. cloud execution.
- For “why doesn’t this apply?” / “how do I configure…?” — **docs + architecture first**, then map to **this repo’s** `.cursor/` tree.
- Prefer **version-accurate** answers; Cursor evolves — align with what the docs say now.

## Expected layout (agents dev repo)

Adapt names to what actually exists in the tree you see:

| Area | Typical path | Role |
| ---- | ------------ | ---- |
| Agent prompts | `.cursor/agents/*.md` | Per-agent or per-stage instructions (frontmatter `name`, `description`) |
| Rules | `.cursor/rules/*.mdc` | Project rules; optional `alwaysApply` / globs |
| Skills | `.cursor/skills/<skill>/SKILL.md` | When-to-apply + procedures |
| Task artifacts (optional) | `.cursor/tasks/runs/`, `templates/` | Versioned pipeline runs, templates |
| User docs | `README.md` (root), `src/docs/*.md` if present | Human guides (e.g. localized how-tos) |

Keep **cross-links** between skill ↔ rule ↔ agents ↔ docs consistent when you add or rename files.

## Team pipeline pattern (if present in this repo)

Some agents-dev repos include a **multi-stage pipeline** (analyst → architect → …). If these paths exist, use them as the contract. Paths below are typical when `.cursor` sits at the repo root; **in this repository** the same layout lives under **`src/.cursor/`**.

- Skill: `.cursor/skills/team-orchestrator/SKILL.md` (here: `src/.cursor/skills/team-orchestrator/SKILL.md`) — **team pipeline contract** for every stage (I/O, `full` / `fast`, `40-plan.plan.md` format); skill id `team-orchestrator` is the folder name, not “orchestrator-only.” Optional `Task` orchestration is documented there too.
- Rule: `.cursor/rules/team-orchestrator.mdc` (here: `src/.cursor/rules/team-orchestrator.mdc`) — short matrix and pointers.
- Per-stage agents: `.cursor/agents/team-pipeline-*.md` (here: under `src/.cursor/agents/`) — inputs, outputs, gates, `manifest.json` keys.
- Artifacts: `.cursor/tasks/runs/…`, `LATEST`, `templates/` (here: `src/.cursor/tasks/…`). Planner output is **`40-plan.plan.md`** (Cursor Plan-style: YAML frontmatter + body, `.plan.md` suffix).
- User guide (RU, team pipeline): root **`README.md`** in this repo; technical index: `src/.cursor/tasks/README.md`.

If this repository **does not** contain those files yet, treat the above as a **reference pattern** when the user asks to add a similar pipeline.

## Working principles

- **One concern per file** where possible: one agent file per role; rules stay thin and link to skills for depth.
- **Gates**: agent bodies should state required inputs on disk before work; avoid silent skips.
- After structural changes, update **docs** and **AGENTS.md** (this file) so the next session still has full context.
