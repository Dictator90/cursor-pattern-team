# Run artifact templates

Copy these into a new run folder under `.cursor/tasks/runs/<YYYYMMDD>_<slug>_<seq>/` when bootstrapping (then fill or replace as stages complete). **Do not** overwrite these source templates from automation.

| Template | Run file | Stage |
| -------- | -------- | ----- |
| [00-brief.md](00-brief.md) | `00-brief.md` | User / orchestrator before analyst |
| [10-analyst.md](10-analyst.md) | `10-analyst.md` | analyst |
| [20-architecture.md](20-architecture.md) | `20-architecture.md` | architect (full) |
| [30-critical-review.md](30-critical-review.md) | `30-critical-review.md` | critical-reviewer (full) |
| [40-plan.plan.md](40-plan.plan.md) | `40-plan.plan.md` | planner (`.plan.md` = Cursor Plan-style: frontmatter + body) |
| [50-plan-review.md](50-plan-review.md) | `50-plan-review.md` | plan_reviewer (full) |
| [60-implementation-notes.md](60-implementation-notes.md) | `60-implementation-notes.md` | developer |
| [70-code-review.md](70-code-review.md) | `70-code-review.md` | code_reviewer |
| [manifest.example.json](manifest.example.json) | `manifest.json` | all stages (update `stages`) |

See also [../README.md](../README.md) and `.cursor/skills/team-orchestrator/SKILL.md`.
