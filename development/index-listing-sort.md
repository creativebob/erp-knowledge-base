# Сортировка списков (index API)

## Бэкенд

- `IndexDTO::resolveOrderBy(allowedColumns, defaultColumn = 'id', defaultDirection = 'desc')` — если `sort_by` пустой или не из whitelist, возвращается пара `(defaultColumn, defaultDirection)`; иначе `(sort_by, sort_direction)` из запроса.
- Трейт `AppliesListingSort` на `BaseService` → метод `applyListingSort($query, $index, $allowed, $defaultColumn, $defaultDirection)`.
- Исключения по смыслу: меню (`order` + `asc`), позиции прайса (`sort_order` + `asc`), медиа альбома (`sort_order` + `asc`), расписания (`starts_at` + `desc`), валюты/языки (`name` + `asc`), и т.д.

## Фронт

- В `usePagination`: при отсутствии `sort_by` в URL считается `descending: true`, чтобы не уходил ложный `sort_direction: asc`.

## Новые сущности

- Для index добавлять whitelist колонок и вызывать `applyListingSort` (или `resolveOrderBy` там, где нет `BaseService`).
