---
doc_status: current
last_reviewed: 2026-03-19
---

# План: сущность «Производитель» (Manufacturer)

## Имя в системе

| Слой | Значение |
|------|----------|
| UI (RU) | **Производитель** |
| Код (EN) | `Manufacturer`, таблица `manufacturers` |
| Модель | `App\Models\Commons\Manufacturer` |
| API / entity alias | `manufacturers`, префикс **`commons/manufacturers`** |
| Маршруты фронта | **`/commons/manufacturers`** (`index`, `create`, `[id]` с алиасом `…/edit`) |

Не путать с маркетинговым `brands` — это другой домен.

## Backend

- Таблица `manufacturers` (Commons): **тенант** `company_id`, полиморфная связь `manufacturerable_type` / `manufacturerable_id` (`company` | `person` из morphMap), **без колонки `name`** (отображаемое имя — аксессор по `manufacturerable`: компания → `name`, персона → `full_name`). Поля справочника: `code`, `description`, `display`, `system`, авторы, soft deletes. **Страна и сайт не хранятся у производителя** — задаются в карточке компании (или иных данных субъекта) через `manufacturerable`. Поле `code` перед сохранением нормализуется через **`App\Support\Strings\StringNormalizer::nullableTrim`**.
- Уникальность: `(company_id, manufacturerable_type, manufacturerable_id)` и `(company_id, code)` при непустом коде.
- `products.manufacturer_id` — nullable FK на `manufacturers`, `onDelete('set null')`.
- Сортировка списка: **белый список полей в `ManufacturerService::ALLOWED_SORT_FIELDS`** (поля `name` в таблице нет; сортировка по имени через колонку не используется).
- CRUD + **`POST commons/manufacturers/lifecycle-create`** (создание карточки компании или персоны в одной транзакции с записью производителя, по образцу Vendor/Customer): `ManufacturerLifecycleService`, `ManufacturerLifecycleCreateRequest`.
- Обычный `POST commons/manufacturers` (store) остаётся для сценариев с уже существующим `manufacturerable`.
- `bulk-delete`, `ManufacturerPolicy`, guard удаления: **Observer `deleting` + проверка в сервисе**, если есть товары с `manufacturer_id`.
- Обновление производителя: `ManufacturerRequest` / `ManufacturerService::update` — **без смены morph и без поля `name`**; карточка компании/персоны обновляется отдельными `PUT` на `business/companies/{id}` и `persons/{id}`.
- `ManufacturerResource`: вычисляемое `name`, вложенный `manufacturerable` (`CompanyResource` / `PersonResource` при загрузке связи).
- Продукт: при сохранении проверяется принадлежность производителя компании товара и **`ManufacturerService::assertManufacturerableBelongsToCompany`**. API производителей: **`commons/manufacturers`** (не в группе `sales`).

## Frontend

- Страницы: `frontend/pages/commons/manufacturers/` — **`create.vue`** (тип `?type=company|person`, `BusinessCompanyForm` / `PersonsForm`, вкладка «Данные производителя»), **`[id].vue`** (редактирование: форма компании/персоны + вкладка полей производителя, сохранение — company/person PUT + manufacturer PUT).
- Компоненты: `commons/manufacturer/list/filters.vue` (без фильтра по стране — страна в карточке компании), **`commons/manufacturer/form/manufacturerDataTab.vue`** (`CommonsManufacturerFormManufacturerDataTab`) — код, описание, флаги.
- **`useManufacturerFormPayload`**: `buildLifecycleCreatePayload`, `buildManufacturerPayload`, `buildCompanyPayload`, `buildPersonPayload` (как у вендора/клиента).
- У **`BusinessCompanyForm`** и **`PersonsForm`** проп **`extraDataTabLabel`** (на экране производителя — «Данные производителя» вместо «Клиентские данные»).
- Селектор: `commons/manufacturer/form/select.vue` (`CommonsManufacturerFormSelect`) — см. `frontend/reusable-components-registry.md`; в списке API поле **`name`** приходит с бэка (аксессор).
- Форма товара: поле производителя на основной вкладке.
- Список товаров: фильтр по производителю в `sales/catalog/product/list/filters.vue`.
- Для кнопки «Редактировать» в таблице производителей у `CommonsPageGroupButtons` задан `edit-route-prefix="/commons/manufacturers"` (API-путь ≠ UI-путь).

## Порядок внедрения (факт)

1. Миграции (только исходный `create_manufacturers`) + модель + сервис + DTO + lifecycle + контроллер + policy + маршруты + сиды сущности и прав.
2. `php artisan migrate --seed`, проверка API и `POST .../lifecycle-create`.
3. Страницы `commons/manufacturers` + вкладка данных производителя + composable payload.
4. Селектор + интеграция в продукт + фильтр в списке товаров.
5. Feature-тесты: `tests/Feature/Commons/ManufacturerLifecycleCreateTest.php`.
