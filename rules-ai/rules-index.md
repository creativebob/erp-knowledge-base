---
doc_status: current
last_reviewed: 2026-03-24
scope: all
when_to_read: after_project_context
max_read_lines: 160
---

# Rules Index

## Must Follow

- После `project-context.md` выбирать только релевантные rule-файлы под тип задачи.
- Не читать все файлы подряд, если задача узкая.
- При сомнениях по границам задачи сначала уточнять тип изменений.
- Для изменений правил соблюдать организацию из `rules-ai/README.md`.

## Task Routing

- **Новая/изменяемая backend-сущность:** `rules-ai/backend/rules-for-creating-new-entity-guide.md`
- **Новая/изменяемая frontend-сущность (страницы/формы):** `rules-ai/frontend/rules-for-creating-new-entity-by-frontend.md`
- **Только стиль/структура Vue-компонента:** `rules-ai/frontend/rules-for-vue-component.md`
- **Только фильтры списков:** `rules-ai/frontend/filter-components-guide.md`

## Context Seeds

- Инварианты проекта: `rules-ai/project-context.md`
- Индекс KB: [README.md](../README.md)
- Правила KB: [CONTRIBUTING.md](../CONTRIBUTING.md)
- Описательная документация приложения: репозиторий `erp.local`, каталог `docs/`

## Anti-Patterns

- Читать длинные гайды при маленькой правке, где нужен только style-rule.
- Копировать один и тот же пункт в несколько rule-файлов.
- Добавлять новые правила вне `rules-ai/` без явной причины.
