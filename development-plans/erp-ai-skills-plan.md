Ты — senior AI architect и staff-level engineer.
Нужно спроектировать enterprise-grade AI Skill System для мультиарендной ERP системы на стеке Laravel + Quasar + MySQL.

# Контекст проекта

ERP система:

* backend: Laravel
* frontend: Quasar/Vue
* database: MySQL
* архитектура: multi-tenant SaaS
* есть AI assistant
* планируется agentic AI architecture
* AI должен работать через tools/MCP/API
* skills должны храниться внутри ERP, а не в IDE

Нужно подготовить ПОЛНЫЙ технический план реализации системы AI Skills.

---

# Основная идея

Каждый tenant (компания) должен иметь:

* собственные AI skills
* собственные workflows
* собственные AI policies
* собственную AI memory
* собственные инструкции для AI assistant

Skills должны:

* храниться в MySQL
* храниться в формате Markdown
* быть editable через UI
* versioned
* auditable
* searchable
* tenant-scoped
* usable runtime AI-агентом

AI assistant должен:

* читать skills
* выбирать релевантные skills
* использовать их во время reasoning
* предлагать улучшения skills
* создавать drafts изменений
* НЕ изменять production skills напрямую

---

# Что нужно подготовить

Подготовь ДЕТАЛЬНЫЙ implementation plan уровня enterprise architecture.

НЕ пиши поверхностно.

Нужен production-grade design.

---

# Нужно раскрыть следующие темы

## 1. Общая архитектура системы

Опиши:

* high-level architecture
* runtime architecture
* agent architecture
* interaction between:

  * ERP
  * AI assistant
  * Skill Engine
  * Memory
  * Tools/MCP
  * Vector Search
  * Policies
  * Planner
  * RAG layer

Нужны:

* схемы
* diagrams
* flows
* lifecycle

---

## 2. Database design

Спроектируй:

* ai_skills
* ai_skill_versions
* ai_skill_runs
* ai_skill_feedback
* ai_skill_embeddings
* ai_policies
* ai_memories
* ai_agent_sessions
* ai_tool_permissions

Нужны:

* SQL schema examples
* indexes
* tenant isolation
* soft deletes
* versioning strategy
* rollback strategy

---

## 3. Skill format

Предложи production-ready Markdown format для skills.

Skill должен поддерживать:

* metadata
* triggers
* tags
* workflow steps
* output format
* escalation rules
* examples
* tool permissions
* evaluation criteria
* safety rules

Пример:

* YAML frontmatter
* markdown body
* structured sections

---

## 4. Runtime AI flow

Опиши:

* как AI получает user request
* как определяется intent
* как выбираются skills
* как работает retrieval
* как skills injectятся в context
* как agent использует tools
* как строится reasoning loop

Нужен пошаговый flow.

---

## 5. Skill Retrieval System

Очень важно.

Опиши:

* semantic search
* embeddings
* vector DB strategy
* hybrid retrieval
* reranking
* context assembly
* token budgeting

Нужны:

* retrieval architecture
* ranking strategy
* performance considerations

---

## 6. Multi-tenant isolation

Критически важно.

Опиши:

* tenant isolation
* embedding isolation
* vector isolation
* memory isolation
* access control
* RBAC
* audit logs
* security boundaries

---

## 7. Skill editing and self-improving AI

Опиши безопасную архитектуру:

AI НЕ должен:

* напрямую менять production skills

AI должен:

* предлагать patches
* создавать drafts
* запускать approval workflow

Нужны:

* patch architecture
* diff strategy
* approval flows
* human-in-the-loop
* rollback
* moderation

---

## 8. Tool/MCP integration

Опиши:

* как tools регистрируются
* как AI получает доступ
* permission system
* sandboxing
* tool execution layer
* auditability
* retries
* error handling

---

## 9. Laravel backend architecture

Нужен production-level design.

Опиши:

* services
* repositories
* queues
* events
* jobs
* pipelines
* domain boundaries
* caching
* observers
* policies
* DTOs
* action classes

Предложи структуру папок.

---

## 10. Quasar frontend architecture

Опиши:

* Skill editor
* Markdown editor
* Diff viewer
* Skill version history
* AI patch approval UI
* Search UI
* Tenant admin UI

Нужны:

* UX recommendations
* page structure
* component hierarchy

---

## 11. AI orchestration

Опиши:

* planner agent
* executor agent
* retrieval agent
* memory manager
* tool manager
* context builder

И как они взаимодействуют.

---

## 12. Observability and monitoring

Нужны:

* logs
* traces
* telemetry
* AI audit logs
* token usage
* hallucination tracking
* evaluation pipelines
* skill effectiveness scoring

---

## 13. Scaling strategy

Опиши:

* horizontal scaling
* queues
* caching
* vector DB scaling
* async orchestration
* streaming
* distributed workers

---

## 14. Security

Критически важно.

Опиши:

* prompt injection protection
* tenant data leakage prevention
* tool sandboxing
* permission boundaries
* AI safety layer
* moderation
* skill validation

---

## 15. Suggested tech stack

Предложи:

* vector DB
* embedding models
* orchestration frameworks
* MCP implementation
* queue systems
* caching
* observability stack

С объяснением плюсов/минусов.

---

# Важные требования

Нужен:

* production-grade architecture
* НЕ toy example
* НЕ simplistic demo
* enterprise-ready design
* scalable approach
* secure approach
* maintainable architecture

---

# Формат ответа

Структурируй ответ как technical architecture document.

Используй:

* diagrams
* tables
* code examples
* SQL examples
* architecture flows
* sequence diagrams
* pseudocode

---

# Дополнительно

В конце:

1. Опиши roadmap implementation phases
2. MVP vs Enterprise features
3. Technical risks
4. Recommended development order
5. Common architecture mistakes
6. Future evolution ideas

---

# Ключевая цель

Нужно спроектировать:

“Self-improving multi-tenant AI Skill Operating System for ERP”

уровня:

* enterprise SaaS
* AI-native ERP
* agentic workflow platform
* organizational intelligence layer
