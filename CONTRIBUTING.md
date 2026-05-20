---
doc_status: current
last_reviewed: 2026-05-20
---

# Как вести Knowledge Base

## Что переносим сюда

- Правила и чеклисты для разработки (`rules-ai/`, `frontend/` паттерны).
- ADR и кросс-сущностные техконтракты (`adr/`, `architecture/`).
- ERD и заметки по схеме (`erd/`).
- Планы, бэклоги, промпты (`development-plans/`, `prompts/`).

## Что остаётся в `erp.local/docs`

- Описание доменов, API-контракты модулей, operational runbooks.
- Auth, permissions, infrastructure, programs.
- OpenAPI (`erp.local/docs/api/`).

При сомнении — не переносить без явного решения; зафиксировать вопрос в PR или issue.

## Куда класть новые материалы

1. **ADR** → `adr/` (`ADR-NNN-topic.md` или `topic-adr.md`).
2. **Техпаттерны кода** → `architecture/`.
3. **Правила для AI/разработчиков** → `rules-ai/` (см. [rules-ai/README.md](./rules-ai/README.md)).
4. **Промпты** → `prompts/`.
5. **Планы** → `development-plans/`.
6. **ER Canvas** → `erd/{domain}.canvas` + обновить [erd/README.md](./erd/README.md).
7. **UI-паттерны** → `frontend/`; при новом shared-компоненте — [reusable-components-registry.md](./frontend/reusable-components-registry.md).
8. **Бизнес-процессы (для RAG)** → `workflows/` — только по согласованию.

## Frontmatter

```yaml
---
doc_status: current          # current | needs_review | deprecated | historical
last_reviewed: YYYY-MM-DD
---
```

## Ссылки

- Внутри KB — относительные пути без префикса `docs/` (например `rules-ai/project-context.md`).
- На описательную документацию приложения — явная отсылка: репозиторий `erp.local`, путь `docs/...` (в тексте, не как битая относительная ссылка).

## Индекс

[README.md](./README.md) · [STRUCTURE.md](./STRUCTURE.md)
