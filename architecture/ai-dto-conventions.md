---
doc_status: current
last_reviewed: 2026-05-19
---

# AI: конвенции DTO и объектов turn-пайплайна

Контракты в `app/DTO/AI/` **намеренно разделены по ролям**. Не унифицировать всё под Spatie `Data` — для orchestration и gateway это лишнее и вредно.

## Три роли объектов

| Роль | Именование | Базовый тип | Откуда данные | Примеры |
|------|------------|-------------|---------------|---------|
| **API / CRUD** | `Create*`, `Update*`, `*Filters` | `extends Data` | HTTP Request → `::from($request->validated())` | `UpdateAiAgentProfileDTO`, `CreateAiSkillDTO` |
| **Orchestration command** | `AiTurnRequestDTO` (turn command) | `final readonly` | PHP: controller, listener, job dispatch | `AiTurnRequestDTO` |
| **Turn metadata** | `AiTurnContextDTO` | `final readonly` | `AiTurnContextDTO::fromRequest($request)` | workflow, task, parent run |
| **Turn state** | `AiTurnSessionDTO` | mutable class | `AiTurnBootstrap` + enrichers на один turn | модели, tokens, tools bundle |
| **Turn result** | `AiTurnOutcomeDTO` | `final readonly` | orchestrator / direct reply / LLM loop | body, tokens, direct skill meta |
| **Gateway value objects** | `Llm*DTO` | `final readonly` | gateway, context builder | `LlmMessageDTO`, `LlmChatRequestDTO` |
| **Skill runtime** | `SkillExecutionContextDTO` | `final readonly` | tool executor (in-process) | actor, company, branch |

## Слой orchestration (п.6)

Логический слой **Commands / pipeline state** реализован так:

```text
app/Services/AI/Orchestration/     — поведение (dispatcher, orchestrator, lifecycle)
app/DTO/AI/                        — API Data DTO + turn command/metadata
app/DTO/AI/Turn/                   — state + result одного turn
app/DTO/AI/Gateway/                — контракт LLM-провайдера
```

- **Command (очередь):** `AiTurnRequestDTO` → `ProcessAiAgentTurnJob` → `AiTurnDispatcher`.
- **Metadata:** `AiTurnContextDTO` — только поля, дублирующие command; собирается через `fromRequest()`, не вручную в нескольких местах.
- **State:** `AiTurnSessionDTO` — живёт в памяти одного `processTurn`; **не** сериализуется в queue.
- **Result:** `AiTurnOutcomeDTO` → `AiTurnLifecycle::complete()`.

Отдельная папка `app/AI/Commands/` не требуется, пока команда одна (`AiTurnRequestDTO`) и команда явно задокументирована.

## Queue-safe контракт (п.2)

`AiTurnRequestDTO` **можно** класть в `ProcessAiAgentTurnJob`: только `int`, `string`, `bool`, `AiRunTriggerEnum`.

**Запрещено** добавлять в command: Eloquent-модели, `Collection`, замыкания, `SkillExecutionContextDTO` (содержит `User`).

Проверка: `Tests\Unit\AI\AiTurnRequestDtoSerializationTest`.

## Spatie Data — когда использовать

- Index/create/update API, фильтры, cursor pagination.
- Валидация: **один источник** — Form Request **или** атрибуты на `Data`, не дублировать оба без нужды.

## `final readonly`

Все internal value object и command/metadata/outcome/gateway DTO — `final readonly class`, кроме mutable `AiTurnSessionDTO`.

## Связанные документы

- `erp.local/docs/domains/ai/README.md` — операционный гайд AI (агенты, skills, чат)
- [company-scoped-visibility-and-templates.md](./company-scoped-visibility-and-templates.md) — platform skills, tenant visibility
- [../development-plans/ai-assistent-plan.md](../development-plans/ai-assistent-plan.md) — дорожная карта AI
