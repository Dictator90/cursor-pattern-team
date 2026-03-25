# Контекст для ИИ: паттерн «команда» (team pipeline)

Этот файл задаёт контекст для работы **по** многостадийному пайплайну в репозитории, где каталог Cursor лежит рядом с этим файлом (в данном репозитории это `src/.cursor/`). Он **не** описывает разработку самого паттерна как набора правил Cursor — только использование стадий, агентов и артефактов.

## Как читать пути

| Как открыт проект в Cursor | Агенты и `.cursor` |
| ------------------------- | ------------------- |
| Корень репозитория (родительская папка `src`) | `src/.cursor/agents/…`, `src/.cursor/skills/…`, `src/.cursor/tasks/…` |
| Корень workspace = папка `src` | `.cursor/agents/…`, `.cursor/skills/…`, `.cursor/tasks/…` |

Дальше в таблицах для краткости указан вариант **от папки `src`** (пути без префикса `src/`).

## Стадии и агенты `team-pipeline-*`

Вызовайте в чате Cursor соответствующего **агента** (по имени из frontmatter). В сообщении по возможности укажите `run=<путь-к-папке-run>` и для планировщика/разработчика — `mode=full` или `mode=fast`, если режим неочевиден. Активный прогон по умолчанию — первая строка файла `.cursor/tasks/runs/LATEST`. **Новая тема/аудит** при том же `LATEST` не должна перетирать старый прогон: см. skill `team-orchestrator` § *Same run vs new run*; при необходимости — `new_run=1` или явный `run=`.

| Файл агента | Роль (кратко) |
| ----------- | ------------- |
| `team-pipeline-analyst.md` | Требования, сценарии, критерии приёмки → `10-analyst.md` из `00-brief.md`. Код и архитектура продукта не пишет. |
| `team-pipeline-architect.md` | Архитектура → `20-architecture.md` из `10-analyst.md`. Код продукта только для чтения при необходимости. |
| `team-pipeline-critical-reviewer.md` | Сверка аналитики и архитектуры → `30-critical-review.md`. Перед планом, режим full. |
| `team-pipeline-planner.md` | План работ → `40-plan.plan.md` (формат как у Cursor Plan: YAML + тело). Режимы full / fast. |
| `team-pipeline-plan-reviewer.md` | Ревью плана → `50-plan-review.md`. Обычно full, перед разработкой. |
| `team-pipeline-developer.md` | Реализация по плану, заметки → `60-implementation-notes.md`. |
| `team-pipeline-code-reviewer.md` | Ревью кода после разработки → `70-code-review.md`. |
| `team-pipeline-orchestrator.md` | Порядок стадий, прогоны, подсказки; **не** заменяет остальных агентов. |

Расположение файлов агентов: `.cursor/agents/team-pipeline-*.md` (от корня `src`).

**Полнота артефактов:** markdown-файлы прогона (`10-` … `70-`) должны содержать **тот же полный текст**, что и развёрнутый ответ в чате для этого вызова, а не краткую выжимку только в файле. См. skill `team-orchestrator` § *Artifact completeness (all pipeline markdown outputs)*.

## Контракт пайплайна, правило, артефакты

- **Полный контракт стадий** (что читать/писать на каждом шаге, full/fast, формат `40-plan.plan.md`): [.cursor/skills/team-orchestrator/SKILL.md](.cursor/skills/team-orchestrator/SKILL.md). Идентификатор skill — имя папки `team-orchestrator`; документ относится ко **всем** стадиям, не только к оркестратору.
- **Правило проекта** (краткая матрица и отсылки): [.cursor/rules/team-orchestrator.mdc](.cursor/rules/team-orchestrator.mdc).
- **Прогоны и указатель LATEST**: [.cursor/tasks/README.md](.cursor/tasks/README.md).
- **Шаблоны файлов прогона**: [.cursor/tasks/templates/](.cursor/tasks/templates/) (в т.ч. `40-plan.plan.md`).
