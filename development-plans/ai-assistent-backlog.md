---
doc_status: current
last_reviewed: 2026-05-19
---

# AI-платформа: бэклог будущих задач

Документ фиксирует **отложенные** и **не начатые** работы по AI-платформе. Источник истины по архитектуре, фазам A–I и трекеру «сделано» — [ai-assistent-plan.md](ai-assistent-plan.md).

Статусы пунктов бэклога:

| Статус | Значение |
|--------|----------|
| `open` | Не начато, в очереди |
| `planned` | Есть отдельный план или детальная спецификация |
| `deferred` | Осознанно отложено после MVP |

---

## Приоритет P1 — платформа и продукт

| ID | Тема | Статус | Описание | Связь с планом |
|----|------|--------|----------|----------------|
| **BL-01** | **Streaming LLM в drawer** | `open` | Потоковая отдача токенов от провайдера (SSE/WebSocket/Reverb), UI «печатает по частям». Сейчас: полный ответ после turn; Reverb доставляет только готовое сообщение. Сложность выше при tool loop (stream только финального текста). | [ai-assistent-plan.md](ai-assistent-plan.md) — «Не в scope MVP»; фаза C (Reverb) — только готовые сообщения |
| **BL-02** | **Полная observability** | `open` | Структурированные логи `ai.run.started` / `completed` / `failed`, `ai.tool_call.*` (duration, tokens, tenant, workflow_id); опционально метрики (p95, error rate). Сейчас: ops UI `ai/runs` + warning-логи `ai.llm.*` в gateway. | Фаза I закрыла ops UI; observability — отдельно |
| **BL-03** | **Фаза H — workflow steps** | `open` | Запись `ai_workflow_steps`, lifecycle workflow (`completed`/`failed`), `tasks.workflow_id` при `tasks.create`, multi-step сценарии. Сейчас: `ai_workflows` + `workflow_id` на runs без шагов. | ADR workflow_id в [ai-assistent-plan.md](ai-assistent-plan.md) |
| **BL-04** | **Фаза G — skills: лиды** | `planned` | `leads.count_on_date`, active lead, handlers + seeder + тесты. | [ai-leads-skills-plan.md](ai-leads-skills-plan.md) |
| **BL-05** | **Фаза F — Memory / RAG** | `deferred` | Векторное хранилище (не MySQL), интерфейс `Memory/`, подключение к контексту turn. | Фаза F в [ai-assistent-plan.md](ai-assistent-plan.md) |
| **BL-06** | **Instruction Skills** | `open` | IS-1 MVP **готово** (2026-05-19). Далее: IS-2 tool filter, IS-3 governance. | [ai-instruction-skills.md](ai-instruction-skills.md) |

---

## Приоритет P2 — каналы и UX

| ID | Тема | Статус | Описание | Связь с планом |
|----|------|--------|----------|----------------|
| **BL-10** | Human DM (`employee_dm`) | `deferred` | Чат сотрудников: `kind = employee_dm`, API и UI. Enum/миграция уже есть. | «Не в scope MVP» |
| **BL-11** | OpenClaw как channel ingress | `deferred` | Отдельный ingress рядом с drawer/Telegram; credential `openclaw` в LLM filter. | «Не в scope MVP» |
| **BL-12** | Idempotency для Telegram | `open` | `client_message_id` / mapping на `external_id` для повторов webhook. Drawer уже: `client_message_id`. | Фаза I.2 (drawer) |
| **BL-13** | UI conversations / messages (admin) | `deferred` | Отдельные index-страницы (пункты меню закомментированы). Ops runs — сделано (I.3). | Меню в `default.vue` |

---

## Приоритет P3 — расширения платформы

| ID | Тема | Статус | Описание | Связь с планом |
|----|------|--------|----------|----------------|
| **BL-20** | `http_internal` handler type | `deferred` | Skills с вызовом internal API вместо PHP handler. | «Не в scope MVP» |
| **BL-21** | Generic direct-reply из конфига skill | `open` | Вместо отдельного strategy class на каждый shortcut — конфиг в `ai_skills`. | Orchestrator DirectReply |
| **BL-22** | E2E async turn | `open` | Тест: job → assistant message в БД + broadcast (сейчас unit/feature частично). | Фаза B async |
| **BL-23** | Алерты по failed runs | `open` | Уведомления админу при `ai_runs.status = failed` (зависит от BL-02). | После observability |

---

## Явно вне бэклога (уже сделано)

Не дублировать здесь — см. трек в [ai-assistent-plan.md](ai-assistent-plan.md):

- Фазы A–E, I.1–I.3 (catalog, idempotency, ops runs)
- OpenAI `.env` fallback (`OPENAI_API_KEY`, `EXTERNAL_CREDENTIAL_PLATFORM_POOL_TENANT_ENV_FALLBACK`)
- Декомпозиция orchestrator (`Turn/`, `DirectReply/`, `Llm/`)

---

## Как добавлять пункты

1. Новый пункт — строка в таблице с ID `BL-XX`, статус `open` | `planned` | `deferred`.
2. Если появляется отдельный план (как лиды) — статус `planned` + ссылка на файл в `docs/development-plans/`.
3. После реализации — удалить из бэклога и отметить `[x]` в [ai-assistent-plan.md](ai-assistent-plan.md).

---

## Связанные документы

- [ai-assistent-plan.md](ai-assistent-plan.md) — дорожная карта, ADR, трек исполнения
- [ai-leads-skills-plan.md](ai-leads-skills-plan.md) — детализация BL-04
- [ai-instruction-skills.md](ai-instruction-skills.md) — детализация BL-06
- [internal-messaging-chat-plan.md](internal-messaging-chat-plan.md) — drawer UI
- [tenant-external-credentials-plan.md](tenant-external-credentials-plan.md) — LLM keys и platform pool
