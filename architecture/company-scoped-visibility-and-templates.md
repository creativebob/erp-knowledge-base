---
doc_status: current
last_reviewed: 2026-04-03
---

# Видимость company-scoped записей и межтенантные шаблоны

Документ описывает сквозную модель: **квадранты записи** (`CompanyScopedRecordQuadrant`), **глобальные шаблоны** (`company_id = null`, не платформенные), **слой видимости** (`CompanyScopedVisibility`, `withTemplates()`), **политики** и **отдельные права на правку/удаление шаблонов**.

Связанные материалы:

- Права `update templates` / `delete templates`: `erp.local/docs/permissions/permissions-architecture.md` (блок «Межтенантные шаблоны»).
- Контекст компании и резолвер: [platform/site-context-resolution-adr.md](../platform/site-context-resolution-adr.md).
- Сущность **`Company`** (организация) **не** входит в этот стек; отдельно: [company-entity-access-adr.md](company-entity-access-adr.md).

---

## 1. Зачем это нужно

Часть сущностей хранит строки **в контексте компании** (`company_id` заполнен) и строки **межтенантных шаблонов** (`company_id = null`, `is_system = false`), плюс **платформенные** глобальные (`company_id = null`, `is_system = true`).

Требования:

- Индекс и `show` по `id` должны **не расходиться**: шаблон, видимый в списке, должен открываться по идентификатору.
- Глобальный scope **`HasCompany`** по умолчанию ограничивает выборку текущей компанией — для списков и `find`, где нужны шаблоны, используется **явное расширение** видимости, а не изменение глобального scope «для всех запросов».
- Политика **view/update/delete** должна быть согласована с SQL-видимостью и с матрицей прав (в т.ч. отдельные права на **редактирование эталонных шаблонов**).

---

## 2. Квадранты: `CompanyScopedRecordQuadrant` и трейт `HasCompanyScopedQuadrant`

Перечисление [`app/Enums/Tenant/CompanyScopedRecordQuadrant.php`](../../app/Enums/Tenant/CompanyScopedRecordQuadrant.php) классифицирует запись по паре **`company_id`** и **`is_system`**:

| Квадрант | Условие | Смысл |
|-----------|---------|--------|
| `GlobalPlatform` | `company_id === null` и `is_system` | Платформа (глобально, системная) |
| `GlobalTemplate` | `company_id === null` и не `is_system` | Межтенантный **шаблон** (эталон для тенантов) |
| `TenantCustom` | `company_id` задан, не `is_system` | Кастомная запись тенанта |
| `TenantSystem` | `company_id` задан, `is_system` | Системная запись в рамках тенанта |

Трейт [`HasCompanyScopedQuadrant`](../../app/Models/Traits/HasCompanyScopedQuadrant.php) добавляет метод **`companyScopedQuadrant()`** на модель. Его используют политики (например, распознавание строки-шаблона для отдельных прав `update templates` / `delete templates`).

---

## 3. Глобальный scope компании (`HasCompany`)

Трейт [`HasCompany`](../../app/Models/Traits/HasCompany.php) вешает scope **`company`**: при наличии контекста компании запросы по умолчанию фильтруются по `company_id`.

**Шаблоны** (`company_id = null`) при таком scope в «обычном» `Model::query()->find($id)` **не попадают** в выборку. Поэтому для списков и загрузки по `id`, где шаблоны должны быть доступны по продуктовым правилам, используется слой ниже — **`CompanyScopedVisibility`** и **`withTemplates()`**, а не раздувание глобального scope на все запросы.

---

## 4. Сервис видимости: `CompanyScopedVisibility`

Класс [`app/Services/Visibility/CompanyScopedVisibility.php`](../../app/Services/Visibility/CompanyScopedVisibility.php) — **единая точка** построения `Builder` для:

- **списков** — `listingQuery($modelClass, ?User)` (используется из `BaseService::companyScopedIndexQuery`);
- **одной записи** — `find($modelClass, $id, ...)` (используется из `BaseService::companyScopedFind`);
- **проверки id вне HTTP-контекста** — `existsForCompanyScope($modelClass, $companyId, $id)` (тот же предикат, что у «тенант + шаблоны» без `view any`, для явно заданной компании).

Логика ветвления:

- При наличии **`view any {alias}`** список/поиск расширяется: **своя компания + все глобальные строки** (`company_id IS NULL`), в том числе платформенные (`is_system`) и **без** дополнительного SQL из `RestrictsTenantGlobalTemplateRows`. Это намеренно шире, чем обычный тенантский листинг (полный обзор глобальных данных для роли с `view any`).
- Без **`view any`** применяется предикат **«тенант + видимые шаблоны»**: своя компания или шаблон с `company_id IS NULL`, `is_system = false` (если колонка есть), плюс при необходимости `RestrictsTenantGlobalTemplateRows`.

**Без пользователя (`$user === null`):**

- `listingQuery` — возвращает `Model::query()` **со** стандартным глобальным scope `company` (шаблоны в списке не видны; поведение узкое, как у «обычного» запроса без сервиса).
- `find` — одна запись **со** scope `company` на билдере; шаблоны по id через этот путь не откроются.
- `constrainQueryLikeListing(..., null)` — снимает scope `company` и накладывает **только** «тенант + шаблоны» для `CompanyResolver::currentCompanyId()` (без ветки `view any` и без утечки строк других тенантов). Если контекста компании нет — остаются только применимые глобальные шаблоны по тем же правилам, что и при `company_id === null` в `applyNonViewAnyTenantListingWheres`.

Кэширование алиаса сущности для проверки `view any` — внутри сервиса (`entities` + fallback `listingPermissionEntityAlias()`).

---

## 5. Локальный scope `withTemplates()`

Трейт [`ScopesTenantTemplates`](../../app/Models/Traits/ScopesTenantTemplates.php) добавляет **`scopeWithTemplates`**, который делегирует в `CompanyScopedVisibility::applyWithTemplatesScope` — снимает scope компании и накладывает предикат **без ветки `view any`**.

Для **индекса** нельзя заменить `companyScopedIndexQuery` одним `::with Templates()`: у индекса должна учитываться ветка **`view any`**. Внутри `listingQuery` для ветки без `view any` используется **`withTemplates()`**, чтобы не дублировать условия в коде.

На текущий момент трейт подключён у моделей **`Position`** и **`Role`**.

---

## 6. Роли: уточнение глобальных шаблонов

Интерфейс [`RestrictsTenantGlobalTemplateRows`](../../app/Services/Contracts/Visibility/RestrictsTenantGlobalTemplateRows.php) позволяет модели уточнить, какие глобальные несистемные строки видны тенанту **без `view any`** (в SQL и в политике согласуются правила для `Role`).

Модель **`Role`** реализует контракт и фильтрует по **`template_purpose`** (в т.ч. значение **`tenant_shared_template`** в enum [`RoleGlobalTemplatePurpose`](../../app/Enums/Users/RoleGlobalTemplatePurpose.php); чертежи провижининга в общий «каталог» шаблонов тенанта не попадают).

---

## 7. Политики и права на шаблоны

Трейт [`HandlesCompanyScopedEntityAccess`](../../app/Policies/Concerns/HandlesCompanyScopedEntityAccess.php):

- Для записей в квадранте **`GlobalTemplate`** (через `companyScopedIsGlobalIntertenantTemplate`) на **update** дополнительно требуется **`update templates {alias}`** (после базовой проверки **`update {alias}`**).
- Для тех же записей на **delete / restore / force delete** дополнительно требуется **`delete templates {alias}`**.

Так пользователь с **`view any`** и **`update`**, но **без** `update templates`, **не меняет** эталонные шаблоны.

Перечень действий и имён прав в БД — общий пайплайн **Action → Entity → PermissionSeeder**; см. `erp.local/docs/permissions/permissions-architecture.md`.

---

## 8. Базовый сервис и вызовы из доменных сервисов

[`BaseService`](../../app/Services/BaseService.php):

- **`companyScopedIndexQuery($modelClass)`** → `CompanyScopedVisibility::listingQuery`;
- **`companyScopedFind($modelClass, $id, $with)`** → `CompanyScopedVisibility::find`.

Примеры: [`PositionService`](../../app/Services/Business/Managment/PositionService.php), [`RoleService`](../../app/Services/Users/RoleService.php).

---

## 9. Расширение на новые сущности

1. Модель с **`HasCompany`** + при необходимости **`HasCompanyScopedQuadrant`**.
2. Подключить **`ScopesTenantTemplates`**, если в индексе/show участвуют межтенантные шаблоны.
3. В **`EntitiesTableSeeder`** для сущности уже должны быть действия (в т.ч. при необходимости **`update templates`** / **`delete templates`**).
4. Политика на базе **`HandlesCompanyScopedEntityAccess`** (или эквивалентная логика для `GlobalTemplate`).
5. Индекс/show сервиса — через **`companyScopedIndexQuery` / `companyScopedFind`** (или эквивалентный вызов `CompanyScopedVisibility`).

---

## 10. Тесты

Регрессии по политикам и шаблонам: **`tests/Feature/Business/Managment/PositionQuadrantPolicyTest.php`**, **`tests/Feature/Users/RoleQuadrantPolicyTest.php`** (в т.ч. сценарии без `update templates` / `delete templates`).

Матрица SQL-видимости сервиса: **`tests/Feature/Services/Visibility/CompanyScopedVisibilityTest.php`** (`existsForCompanyScope`, `constrainQueryLikeListing` без пользователя при заданном `TenantContext`).

---

## 11. Краткое резюме

| Компонент | Назначение |
|-----------|------------|
| `CompanyScopedRecordQuadrant` | Классификация записи по `company_id` / `is_system` |
| `HasCompanyScopedQuadrant` | Метод `companyScopedQuadrant()` на модели |
| `CompanyScopedVisibility` | Единая видимость для списка и `find`, ветка `view any`, `existsForCompanyScope` |
| `withTemplates()` | Предикат «тенант + шаблоны» без ветки `view any` |
| `RestrictsTenantGlobalTemplateRows` | Уточнение SQL/политики для глобальных ролей |
| `update templates` / `delete templates` | Отдельные права на изменение эталонных шаблонов |

После изменения сидеров **`ActionsTableSeeder`** / **`EntitiesTableSeeder`** нужно прогонять **`PermissionSeeder`** (или полный цикл миграций/сидов по регламенту проекта), чтобы новые строки прав появились в таблице `permissions`.
