Для enterprise ERP + AI платформы такая структура knowledge base — это уже не просто “папка с заметками”.

Это:

> Engineering Operating System.

Разберу подробно каждую директорию и зачем она нужна именно вам.

---

# Что такое ERD

# ERD = Entity Relationship Diagram

Диаграмма структуры базы данных.

---

# ERD показывает:

* таблицы
* поля
* связи
* foreign keys
* cardinality
* зависимости

---

# Пример

```text id="tb5b7r"
users
 ├── id
 ├── company_id
 └── email

companies
 ├── id
 └── name

users.company_id
    ↓
companies.id
```

---

# Зачем это вам

У вас:

* ERP
* multi-tenant
* CRM
* HR
* AI
* workflows

Схема БД быстро станет огромной.

ERD помогает:

* понимать связи
* проектировать модули
* объяснять AI архитектуру
* анализировать impact changes

---

# Что хранить в `/erd`

---

# Примеры

```text id="l9v0b6"
erd/
 ├── crm-core.md
 ├── hr-domain.md
 ├── ai-platform.md
 ├── tenant-model.md
 └── billing.md
```

---

# Внутри

* Mermaid ER diagrams
* draw.io
* Excalidraw
* screenshots
* DB notes

---

# Очень важно

ERD — это:

> документация структуры данных

---

# Что такое ADR

# ADR = Architecture Decision Record

Это одна из самых мощных практик enterprise разработки.

---

# ADR отвечает на вопрос:

> Почему было принято архитектурное решение?

---

# Не ЧТО сделали.

# А ПОЧЕМУ.

---

# Пример ADR

## Название

```text id="z2t0uv"
ADR-001-use-modular-monolith.md
```

---

# Внутри

```markdown id="1l8d1s"
# ADR-001
# Use Modular Monolith Instead of Microservices

## Status
Accepted

## Context
ERP is developed by solo developer.
Need fast iteration speed.

## Decision
Use modular monolith architecture.

## Consequences
+ Faster development
+ Easier deployment
- Future extraction complexity
```

---

# Почему ADR критически важен для вас

Потому что через 1-2 года вы забудете:

* почему выбрали Redis Streams
* почему AI Gateway внутри монолита
* почему не сделали microservices
* почему skills через contracts

---

# ADR — это память архитектора.

---

# Что хранить в `/adr`

---

# Примеры

```text id="m8vfqc"
adr/
 ├── ADR-001-modular-monolith.md
 ├── ADR-002-ai-gateway.md
 ├── ADR-003-multi-tenancy.md
 ├── ADR-004-rag-strategy.md
 └── ADR-005-event-bus.md
```

---

# Теперь про всю структуру

---

# `/architecture`

# Главная архитектура системы

---

# Что хранить

## System design

```text id="59js1f"
architecture/
 ├── system-overview.md
 ├── ai-platform.md
 ├── multi-tenancy.md
 ├── event-bus.md
 ├── queue-system.md
 ├── permissions.md
 └── deployment.md
```

---

# Здесь описывается:

* как устроена ERP
* bounded contexts
* AI platform
* integrations
* modules
* queues
* websocket architecture
* gateway layers

---

# Это:

> high-level architecture

---

# `/ai`

# Всё связанное с AI платформой

---

# Что хранить

```text id="67vqk5"
ai/
 ├── agents/
 ├── skills/
 ├── memory/
 ├── rag/
 ├── providers/
 ├── prompts-strategy.md
 └── instruction-skills.md
```

---

# Здесь:

* AI agents
* skills
* workflows
* tool calling
* memory architecture
* RAG
* embeddings
* prompt orchestration

---

# Это:

> AI subsystem knowledge

---

# `/prompts`

# Prompt engineering layer

---

# Что хранить

```text id="y8idny"
prompts/
 ├── sales-agent.md
 ├── hr-agent.md
 ├── summarize-client.md
 ├── extract-intent.md
 └── lead-qualification.md
```

---

# Здесь:

* system prompts
* templates
* instruction blocks
* extraction prompts
* classifiers
* chain prompts

---

# Очень важно

НЕ хранить prompts хаотично по коду.

---

# `/erd`

# Database architecture

---

# Что хранить

* ER diagrams
* table relations
* tenancy structure
* module schemas
* domain entities

---

# Это:

> data architecture

---

# `/workflows`

# Бизнес-процессы

---

# Это ОЧЕНЬ важная директория для ERP

---

# Что хранить

```text id="n3ml8w"
workflows/
 ├── lead-processing.md
 ├── employee-onboarding.md
 ├── invoice-approval.md
 ├── ai-sales-flow.md
 └── recruitment-pipeline.md
```

---

# Здесь описывается:

## Не код.

## А процессы бизнеса.

---

# Пример workflow

```text id="k0zdvh"
New Lead
  ↓
AI qualification
  ↓
Assign manager
  ↓
Generate proposal
  ↓
Follow-up reminder
```

---

# Это:

> business process layer

---

# `/adr`

# История архитектурных решений

---

# Это:

> память системы

---

# Почему вся эта структура особенно важна для AI

Потому что позже вы сможете:

```text id="6eivj0"
knowledge-base
      ↓
embeddings
      ↓
RAG
      ↓
AI Engineering Assistant
```

---

# Тогда AI сможет понимать:

* архитектуру ERP
* бизнес-процессы
* DB schema
* AI agents
* workflows
* technical decisions

---

# Это уже следующий уровень разработки

Не просто:

* codebase

А:

* semantic engineering system

---

# Как это помогает лично вам

Вы solo developer.

Через год у вас будет:

* огромная ERP
* десятки модулей
* AI agents
* integrations
* workflows

Без knowledge system:

* начнётся хаос
* потеряется архитектура
* AI перестанет понимать проект

---

# А с такой структурой

У вас появится:

```text id="h4ngjd"
Code
+
Architecture
+
Business Context
+
AI Context
+
Decision History
```

---

# И это огромнейшее преимущество

Особенно для AI-assisted enterprise development.
