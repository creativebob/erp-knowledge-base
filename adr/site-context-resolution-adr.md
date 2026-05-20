---
doc_status: current
last_reviewed: 2026-03-26
---

# ADR: Site Context Resolution Contract

## Статус
Accepted

## Контекст
В системе одновременно существуют:
- внутренний ERP-контур (единый системный сайт `erp-system`);
- внешние клиентские контуры (разные сайты, разные домены, API-доступ).

Ранее логика выбора `site_id` и проверка соответствия `company_id`/`site_id`
были распределены по нескольким сервисам, что создавало дублирование и
расхождения поведения.

## Решение
Единая точка резольвинга и валидации `site_id` — `SiteContextResolver`:
- `app/Services/Tenant/SiteContextResolver.php`

Принят следующий контракт:

1. Internal-сценарии (ERP)
- Использовать `resolveInternalSiteId(?int $requestedSiteId, int $companyId): int`.
- Если `requestedSiteId` не передан, используется `erp-system`.
- `erp-system` разрешён в internal-контуре как системный сайт ERP.

2. External-сценарии (public/client API)
- Использовать `resolveExternalSiteId(?int $requestedSiteId, int $companyId): int`.
- `site_id` обязателен.
- Fallback запрещён.
- Несоответствие сайта и компании приводит к `business.user.site_company_mismatch`.

## Интеграция
- Internal:
  - `UserService::create`
  - `UserService::createForExistingPerson`
  - `EmployeeService` (создание доступа на edit без явного `site_id`)
  - `TenantManagementService` (ERP-сайт через resolver)
- External:
  - `Invitation handlers` (accept-flow)
  - `InvitationService` / `InviteCampaignService` при резольвинге текущего сайта

Также для client API включён middleware `resolve-site`:
- `bootstrap/app.php` -> маршрутная группа `api/client/v1`.

## Допустимые точки создания User
Разрешённые:
- `UserService` (основной internal API-контур).
- `Invitations/Handlers/AbstractInvitationTypeHandler` (external acceptance-flow).
- `Auth/RegisteredUserController` (legacy web-flow, временно вне strict external).

Недопустимо:
- Добавлять новые точки `User::create(...)` в сервисах/контроллерах без прохождения
  через данный контракт site/company context.

## Последствия
Плюсы:
- единая и предсказуемая логика `site_id`;
- отсутствие fallback-магии во внешнем контуре;
- меньше дублирования и проще сопровождение.

Компромисс:
- legacy `/register` оставлен без перевода на strict external в рамках текущей итерации,
  чтобы избежать ломающих изменений web-auth контура.
