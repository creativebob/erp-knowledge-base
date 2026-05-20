---
doc_status: current
last_reviewed: 2026-05-18
---

# План: чат с AI-ассистентом в ERP (drawer)

_Срез M.* для drawer выполнен; дальнейшее развитие — [ai-assistent-plan.md](ai-assistent-plan.md)._

> **Мастер-план платформы:** [ai-assistent-plan.md](ai-assistent-plan.md) (gateway, skills в БД, `ai_*`, фазы A–F).  
> Этот документ — **продуктовый срез фазы A**: UI и API переписок в drawer.

## Цель

MVP: правый drawer (Quasar `QChatMessage`), персистентные треды с ассистентом, отправка сообщений, ответ через **оркестратор + LLM** (фаза B). Reverb — доставка новых сообщений в открытый drawer.

Перспектива: human DM (`kind = employee_dm`), Telegram/OpenClaw через `ai_conversations.channel`.

## Зависимости

- [tenant-external-credentials-plan.md](tenant-external-credentials-plan.md) — **готово** (`llm_credential_id` в профиле агента для фазы B).
- [ai-assistent-plan.md](ai-assistent-plan.md) — схема `ai_*`, skills, gateway.

## Принципы

- Таблицы и API: префикс **`ai_*`**, маршруты **`/api/ai/...`** (не `internal_*`, не `messaging_*`).
- Код: `App\Services\AI\Conversations\`, контроллеры `App\Http\Controllers\AI\`.
- Не смешивать с `erp.local/docs/domains/communications/communications-foundation.md` (`communications_messages` — только транспорт).
- Мульти-тенантность, Policy, company-scope — [rules-for-creating-new-entity-guide.md](rules-ai/backend/rules-for-creating-new-entity-guide.md).

## Доменная модель (согласовано)

Порядок миграций — в [ai-assistent-plan.md](ai-assistent-plan.md#фаза-a-колонки-mvp-справочник-для-миграций).

| Таблица | Назначение |
|---------|------------|
| `ai_agent_profiles` | Профиль ассистента, `llm_credential_id`, модель, system prompt |
| `ai_conversations` | Тред; `kind` = `assistant_dm` (MVP) |
| `ai_conversation_participants` | Участники (user) |
| `ai_conversation_messages` | Сообщения user / assistant / system |

Skills, runs, tool_calls — общая платформа AI; см. мастер-план.

## UI/UX

- [`useCenterPanels`](../frontend/composables/useCenterPanels.ts), кнопка в [`default.vue`](../frontend/layouts/default.vue).
- Компонент: `frontend/components/commons/layout/center-ai-chat.vue`.
- RBAC: entity для drawer (alias уточнить при реализации, например `ai_conversations`).

## API (MVP)

Middleware: `auth:sanctum`, `refine-tenant-context`, `branches`.

| Метод | Путь | Назначение |
|-------|------|------------|
| GET | `ai/conversations` | Список тредов пользователя |
| POST | `ai/conversations` | Создать тред (или auto default) |
| GET | `ai/conversations/{id}/messages` | История |
| POST | `ai/conversations/{id}/messages` | Сообщение пользователя + stub assistant |

Пагинация истории: **cursor** (согласовать с паттерном проекта при реализации).

## Фазы продукта

| Фаза | Содержание | Связь с мастер-планом |
|------|------------|------------------------|
| MVP | Drawer, `ai_*`, stub-ответ | **Фаза A** |
| Next | Реальный LLM, очередь | **Фаза B** |
| Later | Telegram, human DM | **Фазы E**, `employee_dm` |

## Трек исполнения

### M.0 — Предпосылки

- [x] `external_credentials` и LLM в справочнике (этап 1).

### M.1 — Согласования

- [x] Префикс `ai_*`, пакет `App\Services\AI\Conversations`, API `ai/conversations`.
- [x] Entity `ai_conversations`, права на drawer (`canView('ai_conversations')` в layout).
- [x] Пагинация: cursor (`AiMessageCursorDTO`, meta `next_cursor_id`).

### M.2 — Backend

- [x] Миграции `ai_*` (9 таблиц платформы).
- [x] Модели, DTO, `AiConversationService`, Policies, Controller, Requests, Resources, routes, lang.
- [x] EntitiesTableSeeder + PermissionSeeder; AuthServiceProvider.

### M.3 — Ответ ассистента

- [x] `appendUserMessage` → оркестратор (фаза B; stub заменён).

### M.4 — Frontend

- [x] Drawer `center-ai-chat.vue`, Reverb merge, cursor «Загрузить ранее».

### M.5 — Тесты

- [x] Feature: tenant isolation, policies, LLM (`tests/Feature/AI/`).

## Связанные документы

- [ai-assistent-plan.md](ai-assistent-plan.md)
- [tenant-external-credentials-plan.md](tenant-external-credentials-plan.md)
- [ai-call-tools-preliminary.md](../programs/client-api/ai-call-tools-preliminary.md)
- [project-context.md](rules-ai/project-context.md)
