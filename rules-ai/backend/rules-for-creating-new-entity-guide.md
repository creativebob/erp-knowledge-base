---
doc_status: current
last_reviewed: 2026-03-24
scope: backend_entity
when_to_read: create_or_update_backend_entity
max_read_lines: 260
---

# Backend: создание сущностей

## Must Follow

- Учитывать изоляцию: tenant, branch, site, permissions; если в задаче неочевидно - запросить уточнение.
- Тенантная/филиальная/сайтовая изоляция должна быть централизованной и консистентной с текущей архитектурой.
- Новую сущность регистрировать в `database/seeders/Commons/EntitiesTableSeeder.php`.
- Миграции править через начальные migration-файлы и перекат БД с seed.
- Для каждой таблицы по умолчанию использовать отдельный migration-файл; объединять только тесно связанные сущности.
- Для не-CRUD сервисных методов добавлять короткие поясняющие комментарии.
- Не придумывать новые permission без явного предложения на добавление.

## Task Checklist

- Проверить семантическое имя сущности и отсутствие будущих конфликтов имен.
- Разместить файлы по доменной структуре проекта.
- Продумать индексы и комментарии полей в миграциях.
- Обновить локализацию `lang/ru` для бизнес-сообщений.
- Если сущность должна быть в меню, добавить ссылку в `frontend/layouts/default.vue` с правами.

## Context Seeds

- Tenant изоляция:
	- `app/Services/BaseService.php`
	- `app/Support/TenantContext.php`
	- `app/Services/CompanyResolver.php`
	- `app/Services/Tenant/TenantManagementService.php`
	- `app/Http/Middleware/ResolveCompany.php`
	- `app/Http/Middleware/RefineTenantContext.php`
	- `app/Models/Traits/HasCompany.php`
	- `app/Http/Middleware/EnsureCompanyExternalAccess.php`
- Branch изоляция:
	- `app/Http/Middleware/BranchesPermission.php`
	- `app/Models/Traits/HasBranch.php`
- Site изоляция:
	- `app/Services/SiteResolver.php`
	- `app/Services/Tenant/SiteContextResolver.php`
	- `app/Http/Middleware/ResolveCurrentSite.php`
- API направления:
	- `app/Http/Controllers/PublicApi`
	- `app/Http/Controllers/Internal`
	- `app/Http/Controllers/ClientApi`
- Permissions:
	- `database/seeders/Commons/ActionsTableSeeder.php`
	- `database/seeders/Users/OwnerSeeder.php`
	- `database/seeders/Users/PermissionSeeder.php`

## Anti-Patterns

- Частичная изоляция только в одном слое (например, только middleware без сервисного слоя).
- Новые migration-добавки поверх уже существующей схемы вместо корректировки начальных миграций.
- Дублирование однотипных правил из `project-context.md` в других документах.
