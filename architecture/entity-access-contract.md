---
doc_status: current
last_reviewed: 2026-03-30
---

# Контракт доступа к сущности (Policy + видимость + API)

Единые правила для моделей с публичным API (список, show, мутации). Цель: политики (Spatie), слой видимости в сервисах и контроллеры не расходятся.

## Шаблон строки матрицы (фиксация на сущность)

| Колонка | Что указать |
|---------|-------------|
| **Сущность** | FQCN модели (`App\Models\...`) |
| **Policy** | Класс политики + регистрация в `AuthServiceProvider` |
| **Visibility-слой** | `CompanyScopedVisibility` / `CompanyOrganization` / через родителя / глобальный справочник |
| **Контроллер** | На каждый action: `authorize` после загрузки записи (или `viewAny`/`create` для коллекции) |
| **Исключения** | Отступление от общего паттерна и причина (или «—») |

## Поток

1. **HTTP**: маршрут с `auth:sanctum` (и при необходимости tenant/branch middleware).
2. **Контроллер**: `$this->authorize(...)` на каждый action (для коллекции — `viewAny` / `create`; для записи — после загрузки модели — `view` / `update` / `delete` и т.д.).
3. **Сервис**: список и show через один из паттернов видимости (см. ниже), чтобы выдача совпадала с тем, что разрешено политикой при том же контексте компании/филиала.
4. **Политика**: зарегистрирована в [`app/Providers/AuthServiceProvider.php`](../../app/Providers/AuthServiceProvider.php); алиас прав совпадает с `entities.alias` из сидера (например `bank_accounts`).

## Паттерны видимости в сервисе

| Паттерн | Когда |
|--------|--------|
| `companyScopedIndexQuery` / `companyScopedFind` ([`CompanyScopedVisibility`](../../app/Services/Visibility/CompanyScopedVisibility.php)) | Модель с `HasCompany`, индекс/show согласованы с шаблонами и `view any` |
| `CompanyOrganizationVisibility` | Только сущность **Company** (организационная иерархия), не общий catalog-scoped |
| Видимость через родителя | Пример: **Media** — проверка по модели-владельцу |
| Только глобальный справочник | Редко; явно документировать отсутствие tenant scope |

## Композиция политик

- **Company + шаблоны**: трейт [`HandlesCompanyScopedEntityAccess`](../../app/Policies/Concerns/HandlesCompanyScopedEntityAccess.php).
- **Филиал после company**: [`HasBranchScopedPolicy`](../../app/Policies/Concerns/HasBranchScopedPolicy.php) — порядок как в [`LeadPolicy`](../../app/Policies/Sales/Lead/LeadPolicy.php): сначала `companyScopedCan*`, затем `branchScopedAllows*`.
- **Только `branch_id` без шаблонов tenant** — в матрице сущности явно помечать «только branch», чтобы не подмешивать лишние хуки `companyScoped*`.

## Исключения и осознанные особые случаи

| Сущность | Примечание |
|----------|------------|
| `Company` | [`CompanyPolicy`](../../app/Policies/Business/CompanyPolicy.php), [`CompanyOrganizationVisibility`](../../app/Services/Visibility/CompanyOrganizationVisibility.php) |
| `Media` | [`MediaPolicy`](../../app/Policies/Media/MediaPolicy.php) — доступ через родительскую модель |
| `Bank` | Свой контур (`BankPolicy`), не `HasCompany` у самой модели банка в том же виде, что у счёта |
| `BankAccount` | `HasCompany` + `CompanyScopedVisibility` + `BankAccountPolicy`, алиас `bank_accounts` |

Список whitelist для автопроверки — [`VerifyEntityPolicyMatrixCommand`](../../app/Console/Commands/Quality/VerifyEntityPolicyMatrixCommand.php), команда: `php artisan entity-access:verify-matrix`.

## Чеклист новой сущности с API

1. Запись в [`EntitiesTableSeeder`](../../database/seeders/Commons/EntitiesTableSeeder.php): `alias`, `model`, `actions`, флаги filial/site.
2. Класс **Policy** + регистрация в `AuthServiceProvider::$policies`.
3. **Сервис**: `index` / `show` через `companyScopedIndexQuery` / `companyScopedFind` (или задокументированное исключение).
4. **Контроллер**: все методы с `authorize`; загрузка записи тем же путём, что и `show` сервиса (не «сырой» `find` в обход видимости без причины).
5. Маршруты: `auth:sanctum`, при необходимости `refine-tenant-context` / `branches`.
6. Минимум один тест: 403 при отсутствии права, 404 при id вне видимости (если применимо).

## Связанные документы

- [company-scoped-visibility-and-templates.md](./company-scoped-visibility-and-templates.md)
- [company-entity-access-adr.md](./company-entity-access-adr.md)
- `erp.local/docs/permissions/` — Spatie и роли
