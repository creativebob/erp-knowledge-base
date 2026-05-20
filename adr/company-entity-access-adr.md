---
doc_status: current
last_reviewed: 2026-03-30
---

# ADR: доступ к сущности `Company` (организация) vs company-scoped каталог

## Контекст

В проекте есть единый стек для **тенантных** строк с трейтом [`HasCompany`](../../app/Models/Traits/HasCompany.php): глобальный scope по `company_id`, видимость списков/show через [`CompanyScopedVisibility`](../../app/Services/Visibility/CompanyScopedVisibility.php), политики на [`HandlesCompanyScopedEntityAccess`](../../app/Policies/Concerns/HandlesCompanyScopedEntityAccess.php). Описание в [company-scoped-visibility-and-templates.md](company-scoped-visibility-and-templates.md).

Модель [`Company`](../../app/Models/Business/Company.php) **не** использует `HasCompany`. Поле `company_id` у записи `companies` означает **связь с родительской организацией** (иерархия филиалов / головная компания), а не «тенант, владелец строки каталога» в том же смысле, что у `Product` или `Brand`.

Поэтому применять к `Company` тот же `CompanyScopedVisibility` и `HandlesCompanyScopedEntityAccess` **некорректно**: семантика поля и отсутствие глобального scope `company` на модели привели бы к ошибочным предикатам и ложному ощущению единообразия.

## Решение

1. **Видимость списка и загрузка по id** — отдельный класс [`CompanyOrganizationVisibility`](../../app/Services/Visibility/CompanyOrganizationVisibility.php):
   - при `view any companies` или роли `admin` фильтр списка не сужается;
   - при только `view companies` (без `view any`): строки, где пользователь **автор**, или (для роли `manager` при заданном `user.company_id`) дочерние организации, у которых `companies.company_id` совпадает с контекстом пользователя — в соответствии с политикой.

2. **Политика** — трейт [`HandlesCompanyOrganizationAccess`](../../app/Policies/Concerns/HandlesCompanyOrganizationAccess.php): те же правила `view` / `update` / `delete`, что были в `CompanyPolicy` до выноса (включая роли `admin`, `manager` и права `* any companies`).

3. **`CompanyService`** использует `CompanyOrganizationVisibility` для `index` и `show`, чтобы **индекс и открытие по id не расходились** с `authorize()` на контроллере.

`restore` / `forceDelete` по-прежнему завязаны на роль `admin` в политике (отдельно от тенантного стека).

## Последствия

| Плюсы | Минусы |
|--------|--------|
| Явное разделение «каталог в тенанте» и «реестр организаций». | Отдельный путь в коде: при изменении правил доступа к компаниям править visibility + concern, не `CompanyScopedVisibility`. |
| Нет риска натянуть шаблон `company_id IS NULL` на сущность, где это иерархия. | Для новых разработчиков нужно читать этот ADR, чтобы не копировать паттерн Product на Company. |

## Связанные документы

- [company-scoped-visibility-and-templates.md](company-scoped-visibility-and-templates.md) — к чему применим общий стек.
- `erp.local/docs/permissions/permissions-architecture.md` — матрица прав по сущностям.
