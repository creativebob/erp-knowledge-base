---
doc_status: current
last_reviewed: 2026-05-07
---

# Portfolio MVP Plan

## Цель

Спроектировать и реализовать модуль портфолио в ERP с жесткой консистентностью backend/frontend по архитектуре, структурам файлов, naming и стилю кода.

Модель этапа 1: `Portfolio` -> `PortfolioItem` (с опциональной категоризацией через существующую `PortfolioCategory`).

**Актуальная документация модуля:** обзор домена — [docs/domains/marketing/portfolio.md](../domains/marketing/portfolio.md); контракт REST — [docs/domains/marketing/portfolio-api-contract.md](../domains/marketing/portfolio-api-contract.md).

## Обязательные принципы консистентности

- Повторять зрелые паттерны проекта без произвольных отклонений.
- Backend: полный одинаковый каркас для сущностей (`Controller`, `Request`, `DTO`, `Service`, `Policy`, `Resource`, routes, seeder registration, lang keys).
- Frontend: одинаковая структура страниц (`index/create/[id]`) и компонентов (`form`, `list/filters`) по существующим модулям marketing.
- Tenant isolation обязателен на каждом уровне (query/service/policy/validation).
- API-first: предсказуемые контракты, единый формат ошибок и метаданных.
- AI-agent-ready by design: идемпотентные write-операции, стабильные error `code`, строгий filter/sort whitelist и retry-safe поведение.
- Для index-страниц отклонения от проектного стандарта запрещены: использовать только принятые в проекте компоненты, composables, layout-структуру и порядок блоков.

## Domain Design (Этап 1)

### 1) Portfolio (контейнер)
- Таблица: `portfolios`.
- Поля: `company_id`, `name`, `alias`, `description`, `display`, `is_system`, `author_id`, `editor_id`, timestamps, soft delete.
- Ограничения: уникальный `alias` в рамках `company_id`.

### 2) PortfolioItem (элемент портфолио)
- Таблица: `portfolio_items`.
- Поля: `company_id`, `portfolio_id`, `category_id` (nullable), `name`, `alias`, `description`, `site_url` (nullable), `started_at` / `completed_at` (nullable), `display`, `is_system`, `author_id`, `editor_id`, timestamps, soft delete.
- Обложка: Spatie Media Library на самой модели `PortfolioItem` (коллекция `cover`, `singleFile`), как фото персоны — без колонки `cover_media_id`.
- Ограничения: уникальный `alias` в рамках `portfolio_id`; запрет межтенантных ссылок на `portfolio` и `category`.

### 3) Связи
- `Portfolio` -> `hasMany(PortfolioItem)`.
- `PortfolioItem` -> `belongsTo(Portfolio)`.
- `PortfolioItem` -> `belongsTo(PortfolioCategory)` (nullable).
- Публикация на сайты: существующий `EntityPublication` для обеих сущностей.
- Правило доступности на сайте: `PortfolioItem` считается опубликованным только если одновременно опубликованы `Portfolio` и `PortfolioItem` (both-must-be-on).

## Backend Scope (Этап 1)

### Portfolio
- API:
  - `GET /marketing/portfolios`
  - `GET /marketing/portfolios/{portfolio}`
  - `POST /marketing/portfolios`
  - `PUT /marketing/portfolios/{portfolio}`
  - `DELETE /marketing/portfolios/{portfolio}`
  - `POST /marketing/portfolios/bulk-delete`

### PortfolioItem
- API:
  - `GET /marketing/portfolios/{portfolio}/items`
  - `GET /marketing/portfolios/{portfolio}/items/{item}`
  - `POST /marketing/portfolios/{portfolio}/items`
  - `PUT /marketing/portfolios/{portfolio}/items/{item}`
  - `DELETE /marketing/portfolios/{portfolio}/items/{item}`
  - `POST /marketing/portfolios/{portfolio}/items/bulk-delete`

### Обязательные backend-повторы структуры
- Те же слои и naming-конвенции, что у стабильных marketing сущностей.
- `responseWithEntityMeta()` для индексов.
- Policy с `HandlesCompanyScopedEntityAccess`.
- Регистрация entities/permissions в `database/seeders/Commons/EntitiesTableSeeder.php`.
- Локализация новых бизнес-сообщений в `lang/ru/marketing.php`.

### AI-agent readiness (обязательно в Этапе 1)
- Зафиксировать machine-readable коды доменных ошибок для `Portfolio`/`PortfolioItem` и не завязывать агентную логику на текст `message`.
- Поддержать безопасные повторы write-операций (идемпотентность/retry), включая bulk-операции.
- Для index API гарантировать стабильность названий фильтров, сортировок и метаданных пагинации.
- Обеспечить предсказуемую обработку конфликтов состояния и межтенантных ограничений (`*.cross_tenant_forbidden`).
- Включить `correlation_id`/`source` в наблюдаемость запросов, где это поддержано платформой.
- Добавить явный код ошибки/состояния для правила публикации `both-must-be-on`, чтобы агент не делал эвристик по тексту сообщений.

## Frontend Scope (Этап 1)

### Страницы
- `frontend/pages/marketing/portfolios/index.vue`
- `frontend/pages/marketing/portfolios/create.vue`
- `frontend/pages/marketing/portfolios/[id].vue`
- `frontend/pages/marketing/portfolios/categories/index.vue`
- `frontend/pages/marketing/portfolios/categories/create.vue`
- `frontend/pages/marketing/portfolios/categories/[id]/edit.vue`
- `frontend/pages/marketing/portfolios/[portfolioId]/items/index.vue`
- `frontend/pages/marketing/portfolios/[portfolioId]/items/create.vue`
- `frontend/pages/marketing/portfolios/[portfolioId]/items/[itemId].vue`

### Компоненты
- `frontend/components/marketing/portfolio/portfolio/form.vue`
- `frontend/components/marketing/portfolio/portfolio/list/filters.vue`
- `frontend/components/marketing/portfolio/item/form.vue`
- `frontend/components/marketing/portfolio/item/list/filters.vue`

### Обязательные frontend-повторы структуры
- Повторять существующий каркас index/create/edit из зрелых модулей marketing.
- Повторять паттерн фильтров, bulk-actions, таблиц, pagination/meta.
- Публикационные действия через уже используемые общие компоненты.
- Строго придерживаться правил проекта по Vue-стилю и структуре script/template.
- Для `index.vue` страниц `portfolios` и `items` использовать идентичный composition-подход и те же shared-компоненты, что на эталонных страницах marketing (без новых кастомных паттернов, если существующие покрывают задачу).

## Исключено из Этапа 1

- Универсальные SEO-поля (будущий платформенный модуль).
- Универсальный rich-content/blocks модуль.

Оба направления выделяются отдельными этапами после стабилизации `Portfolio` и `PortfolioItem`.

## Трек исполнения

Статусы:
- `todo` — не начато
- `in_progress` — в работе
- `done` — завершено
- `blocked` — есть блокер

### Этап 1.1 — Domain и backend-контракт
- [x] (`done`) Утвердить финальные поля `portfolios` и `portfolio_items`.
- [x] (`done`) Утвердить ограничения и tenant-правила связей.
- [x] (`done`) Зафиксировать API-контракты для `Portfolio` и `PortfolioItem`.

### Этап 1.2 — Backend implementation
- [x] (`done`) Миграции/модели/DTO/Requests/Services/Policies/Resources.
- [x] (`done`) Роуты и контроллеры с canonical-маршрутизацией: `/marketing/portfolios` и nested `/marketing/portfolios/{portfolio}/items`.
- [x] (`done`) Регистрация entities/permissions + локализация.
- [ ] (`blocked`) Smoke/feature тесты API-критичного контура (в текущем окружении отсутствует sqlite driver для test-раннера).
- [x] (`done`) AI-ready контракт: idempotency/retry, стабильные error-codes, фиксированный filter/sort whitelist.

### Этап 1.3 — Frontend implementation
- [x] (`done`) Страницы и формы `Portfolio`.
- [x] (`done`) Страницы и формы `PortfolioItem`.
- [ ] (`todo`) Интеграция фильтров/list/bulk/publication.
- [ ] (`todo`) Сквозная проверка UX-консистентности с текущими marketing-модулями.

### Этап 1.4 — Приемка консистентности
- [ ] (`todo`) Backend code-style и структурная идентичность эталонным сущностям.
- [ ] (`todo`) Frontend code-style и структурная идентичность эталонным страницам.
- [ ] (`todo`) Проверка tenant-boundary и предсказуемости API-ошибок.
- [x] (`done`) Проверка AI-совместимости API: machine-readable ошибки, безопасный retry, стабильные контракты.
- [ ] (`todo`) Отдельный контроль `index`-консистентности: те же компоненты/composables/структура, что в принятых в проекте marketing-index страницах.

## Этап 2+ (roadmap, позже)
- Универсальный SEO-модуль для всех publishable entities.
- Универсальный rich-content модуль для всех контентных доменов.
