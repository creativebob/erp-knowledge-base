---
doc_status: current
last_reviewed: 2026-03-24
scope: frontend_entity
when_to_read: create_or_update_frontend_entity_pages_forms_components
max_read_lines: 260
---

# Frontend: создание сущностей

## Must Follow

- Соблюдать доменную структуру и принятые naming-паттерны файлов.
- Для плоских сущностей держать структуру консистентной с эталоном `brand`.
- Для древовидных сущностей держать структуру консистентной с эталоном `album/category`.
- Перед созданием нового компонента проверить переиспользуемые компоненты и расширить существующий при возможности.
- Для уведомлений использовать инфраструктуру `useNotify`/realtime по текущим паттернам.
- Если на index-странице выводится телефон, использовать декоратор `frontend/components/commons/decor/phone.vue` (вместо локального форматирования или `usePhone`).
- Общие Vue-правила читать отдельно: `rules-ai/frontend/rules-for-vue-component.md`.

## Task Checklist

- Проверить, что страницы `index/create/[id]` повторяют успешный паттерн аналогичной сущности.
- Проверить, что типы и defaults вынесены в отдельные файлы.
- Проверить, что используется общий `saveButton`/общие form-компоненты там, где это подходит.
- Проверить, что вывод телефона в таблицах index сделан через `CommonsDecorPhone`.
- При добавлении нового переиспользуемого компонента обновить `frontend/reusable-components-registry.md`.
- Для фильтров использовать/проверять `rules-ai/frontend/filter-components-guide.md`.

## Context Seeds

- Эталон плоской сущности (`brand`):
	- `frontend/pages/marketing/brands/[id].vue`
	- `frontend/pages/marketing/brands/create.vue`
	- `frontend/pages/marketing/brands/index.vue`
	- `frontend/components/marketing/brand/form.vue`
	- `frontend/components/marketing/brand/list/filters.vue`
	- `frontend/types/marketing/brand.ts`
	- `frontend/defaults/marketing/brand.ts`
- Эталон древовидной сущности (`album/category`):
	- `frontend/pages/marketing/albums/categories/[id].vue`
	- `frontend/pages/marketing/albums/categories/create.vue`
	- `frontend/pages/marketing/albums/categories/index.vue`
	- `frontend/components/marketing/album/category/form.vue`
	- `frontend/components/marketing/album/category/list/filters.vue`
	- `frontend/types/marketing/album.ts`
	- `frontend/defaults/marketing/album.ts`
- Общие composables:
	- `frontend/composables/usePagination.js`
	- `frontend/composables/useBlank.ts`
	- `frontend/composables/useFormHandler.ts`
	- `frontend/composables/useIndexOperations.js`
	- `frontend/composables/usePermissions.js`
	- `frontend/composables/useAuth.js`
	- `frontend/composables/useNotify.js`
- Общие компоненты:
	- `frontend/components/commons/form/actions.vue`
	- `frontend/components/commons/form/name.vue`
	- `frontend/components/commons/form/emails.vue`
	- `frontend/components/commons/form/phones.vue`
	- `frontend/components/commons/form/photoUpload.vue`
	- `frontend/components/commons/form/unitSelector.vue`
	- `frontend/components/commons/form/digit.vue`
	- `frontend/components/commons/form/alias.vue`
	- `frontend/components/commons/decor`
- Realtime:
	- `frontend/realtime`
	- `frontend/plugins/echo.client.js`

## Anti-Patterns

- Создание плоской структуры именования вместо доменной (`UserListFilters.vue` и подобные).
- Дублирование одинаковой логики form/index вместо выноса в общие composables.
- Игнорирование существующих переиспользуемых компонентов и реестра.

