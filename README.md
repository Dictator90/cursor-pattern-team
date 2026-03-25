# Team pipeline в Cursor — руководство пользователя

Пакет даёт **многостадийный пайплайн** для крупных задач: аналитика → архитектура → план → разработка → ревью. На каждой стадии создаются **файлы-артефакты** в репозитории; каждый новый прогон — **отдельная папка**, старые прогоны не перезаписываются.

## Что вы получаете

- Фиксированные требования, план и заметки по шагам.
- Возможность вызывать стадии **по одной** в Cursor или идти цепочкой.
- Один контракт файлов для вас и для ИИ-агентов Cursor.

## Установка в свой проект

1. Скопируйте каталог **`src/.cursor/`** из пакета в **корень своего репозитория** и переименуйте/положите так, чтобы в проекте появилось дерево **`.cursor/`** (агенты, `tasks/`, `skills/`, при необходимости `rules/`).
2. Ниже в документе пути указаны **как у вас в проекте** — вид `.cursor/...` от корня репозитория.
3. Если вы **просматриваете файлы внутри самого пакета** (до копирования), у путей к `.cursor` добавьте префикс **`src/`** (в поставке пример лежит под `src/.cursor/`).

Дальше везде: **ваш проект** = `.cursor/` в корне открытого в Cursor репозитория.

Подробная схема каталогов и таблица артефактов — в [.cursor/tasks/README.md](src/.cursor/tasks/README.md) (в пакете файл лежит по пути `src/.cursor/tasks/README.md`). Шаблоны для нового прогона — в [.cursor/tasks/templates/](src/.cursor/tasks/templates/).

**Контракт пайплайна** (что на каждой стадии читать и писать): [.cursor/skills/team-orchestrator/SKILL.md](src/.cursor/skills/team-orchestrator/SKILL.md). Имя папки skill — `team-orchestrator`; документ описывает **все** стадии, не только «оркестратор».

## Где что лежит (в вашем проекте)

| Что | Путь |
| --- | ---- |
| Артефакты прогонов | `.cursor/tasks/runs/<YYYYMMDD>_<тема>_<номер>/` |
| Указатель «текущий прогон» | `.cursor/tasks/runs/LATEST` (одна строка — путь к папке run) |
| Шаблоны | `.cursor/tasks/templates/` |
| Агенты по стадиям | `.cursor/agents/team-pipeline-*.md` |
| Правило (по желанию) | `.cursor/rules/team-orchestrator.mdc` |
| Пример контекста ИИ только про пайплайн | в пакете — [src/AGENTS.md](src/AGENTS.md); скопируйте и адаптируйте под себя (см. следующий раздел) |

## Ваш корневой `AGENTS.md` и пайплайн

У вас, скорее всего, уже есть **корневой `AGENTS.md`** с описанием продукта (стек, стиль кода, правила). Пайплайн **не заменяет** его: добавляются стадии, папки `runs/` и отдельные агенты в Cursor.

**Не стоит** вставлять в `AGENTS.md` целиком текст skill и таблицы ввода-вывода — источник правды остаётся в `.cursor/skills/team-orchestrator/SKILL.md`.

**Как совместить:**

1. **Один файл `AGENTS.md`** — добавьте короткий раздел: для крупных задач используете team pipeline; в чате указывайте `run=` и `mode=full` или `mode=fast`; полный контракт — в `.cursor/skills/team-orchestrator/SKILL.md`; имена агентов — `team-pipeline-analyst`, `team-pipeline-planner`, …; при необходимости ссылка на `.cursor/tasks/README.md`.

2. **Два файла** — корневой `AGENTS.md` (продукт) и отдельный, например `.cursor/AGENTS-pipeline.md`, с таблицей стадий, `run`, `LATEST`. Возьмите за основу [src/AGENTS.md](src/AGENTS.md) из пакета, поправьте пути (у вас уже **без** префикса `src/`). В корневом `AGENTS.md` оставьте 1–2 строки со ссылкой и напоминанием в чате подключать `@.cursor/AGENTS-pipeline.md` при работе по стадиям.

3. **В чате** при работе по пайплайну можно добавить в контекст и `@AGENTS.md`, и файл про пайплайн.

## Создание нового прогона (run)

1. Выберите дату `YYYYMMDD` и короткий **slug** темы латиницей с дефисами (например `catalog-filters`).
2. Посмотрите папки в `.cursor/tasks/runs/` с префиксом `YYYYMMDD_<slug>_` и возьмите следующий **двузначный** номер: `01`, `02`, …
3. Создайте папку, например: `.cursor/tasks/runs/20260325_catalog-filters_01`.
4. Скопируйте из `.cursor/tasks/templates/` минимум **`00-brief.md`** и **`manifest.json`** (из `manifest.example.json`, переименуйте в `manifest.json`). Остальные шаблоны — по мере стадий (см. [templates/README](src/.cursor/tasks/templates/README.md)).
5. Заполните `00-brief.md` (цель, контекст, ограничения).
6. В `.cursor/tasks/runs/LATEST` запишите **одну строку** — путь к папке run (от корня репозитория или абсолютный), например:  
   `.cursor/tasks/runs/20260325_catalog-filters_01`

После этого агенты по умолчанию используют этот прогон через `LATEST`, если вы не передадите `run=` вручную.

_В копии пакета без переноса в проект те же шаги выполняйте с префиксом `src/` у путей._

## Ручной режим: явный `run=`

Чтобы агент **не** брал папку из `LATEST`, укажите в **том же сообщении**, что и вызов агента, параметр **`run=<путь-к-папке-run>`** (от корня репозитория или абсолютный путь).

**Важно:** путь разбирается из **всего** текста сообщения, в том числе **после** slash-команды. Сначала всегда ищется `run=`; только если его нет — читается `LATEST`.

Рекомендуемые **прямые слеши** (так проще и на Windows):

```text
/team-pipeline-critical-reviewer Проверь run=.cursor/tasks/runs/20260325_o2k-exchange-audit_01
```

На Windows допустимы **обратные слеши** — агент нормализует путь:

```text
run=.cursor\tasks\runs\20260325_o2k-exchange-audit_01
```

Имя агента в Cursor: **`team-pipeline-…`** (два «p»: *pipe**line***). Опечатка `team-pipline-…` даст другого или несуществующего агента.

Если в сообщении есть **`run=`**, но агент всё равно читает только `LATEST`, обновите файлы `.cursor/` из пакета (контракт в `.cursor/skills/team-orchestrator/SKILL.md`, раздел **Parsing `run=`**).

## Режимы full и fast

**Full** (полный цикл):

`analyst → architect → critical-reviewer → planner → plan_reviewer → developer → code_reviewer`

**Fast** (короче):

`analyst → planner → developer` (в конце по желанию `code_reviewer`)

В **fast** нет отдельной архитектуры и ревью плана; в **`40-plan.plan.md`** в разделе «Assumptions and unknowns» планировщик должен явно описать допущения, неизвестные и границы работ.

**Формат плана:** `40-plan.plan.md` — как план в режиме Plan в Cursor: YAML frontmatter (`name`, `overview`, `todos` с `id` / `content` / `status`, `isProject`), затем тело в Markdown. Суффикс `.plan.md` совпадает с принятым для планов Cursor. Черновик можно обсудить в Plan mode, итог сохраняйте в файл в папке run.

В чате указывайте `mode=fast` или `mode=full`, когда вызываете **team-pipeline-planner** или оркестратор, если режим неочевиден.

## Как работать в Cursor

1. Откройте чат с нужным **агентом** (`team-pipeline-analyst`, `team-pipeline-planner` и т.д.).
2. В сообщении по возможности укажите:
   - **`run=<путь-к-папке-run>`** — явная папка прогона (**приоритет над `LATEST`**); примеры — в разделе **«Ручной режим: явный `run=`** выше;
   - **`mode=full`** или **`mode=fast`** — для планировщика и разработчика, если неясно.
3. Агент проверит **гейт** (наличие входных файлов). Если чего-то нет — остановится и подскажет, какую стадию пройти раньше.

| Файл агента | Когда вызывать |
| ----------- | -------------- |
| `team-pipeline-analyst.md` | есть run и `00-brief.md` |
| `team-pipeline-architect.md` | есть `10-analyst.md` |
| `team-pipeline-critical-reviewer.md` | есть `10` и `20` |
| `team-pipeline-planner.md` | full: есть `10` и `20`; fast: достаточно `10` |
| `team-pipeline-plan-reviewer.md` | есть `40-plan.plan.md` (обычно только full) |
| `team-pipeline-developer.md` | есть `40-plan.plan.md`; в full обычно ещё `50-plan-review.md` (если не снят гейт в `manifest.json`) |
| `team-pipeline-code-reviewer.md` | есть `60-implementation-notes.md` |
| `team-pipeline-orchestrator.md` | нужен порядок стадий или помощь с прогоном |

Цепочку через встроенный `Task` можно настроить по skill; для ручной работы достаточно таблицы выше.

## Если агент «отказал» (гейт)

Сообщение вроде «нет файла X» значит:

- предыдущая стадия не создала артефакт — запустите нужного агента раньше;
- вы в **fast**, а агент ждёт артефакты **full** — укажите `mode=fast` или пройдите стадии;
- неверный **run** — проверьте `LATEST` или явный `run=`.

## Агенты во всех проектах Cursor

Чтобы те же агенты были доступны **глобально**, скопируйте `.cursor/agents/team-pipeline-*.md` из своего проекта в каталог агентов пользователя, например:

`C:\Users\<имя>\.cursor\agents\`

После обновления файлов в проекте имеет смысл обновить и копию в профиле.

## Цепочка артефактов в папке run

Полная таблица — в [.cursor/tasks/README.md](src/.cursor/tasks/README.md). Кратко:

`00-brief.md` → `10-analyst.md` → `20-architecture.md` → `30-critical-review.md` → `40-plan.plan.md` → `50-plan-review.md` → `60-implementation-notes.md` → `70-code-review.md`, плюс `manifest.json` на всех шагах.

## Справочные материалы в пакете

- [src/.cursor/skills/team-orchestrator/SKILL.md](src/.cursor/skills/team-orchestrator/SKILL.md) — контракт стадий.
- [src/.cursor/tasks/templates/README.md](src/.cursor/tasks/templates/README.md) — шаблоны артефактов.
- [src/AGENTS.md](src/AGENTS.md) — пример отдельного контекста «только пайплайн» для копирования в ваш проект.
