---
doc_status: current
last_reviewed: 2026-03-24
scope: frontend_vue
when_to_read: create_or_edit_vue_component
max_read_lines: 140
---


# Vue component rules

## Must Follow

- Не импортировать вручную `ref`, `computed`, `watch`, `onMounted` и другие Vue composables — в Nuxt они автоимпортируются.
- После `<script setup lang="ts">` оставлять одну пустую строку.
- Порядок секций в script: `defineProps` -> `defineEmits` -> composable-хуки -> состояние -> методы -> watchers -> `onMounted`.
- Атрибуты событий (`@...`) размещать в конце списка атрибутов.
- Если у элемента больше двух атрибутов — писать атрибуты вертикально (по одному на строке).
- Типы/интерфейсы выносить в `frontend/types`, если используются более чем в одном месте.
- Для API-вызовов придерживаться `then/catch/finally` (проектный стандарт).
- Не делать предположений о формате API; при сомнениях запрашивать реальный JSON.

## Task Checklist

- Проверить, что нет ручных Vue-импортов composables.
- Проверить единый порядок блоков в `<script setup>`.
- Проверить форматирование template-атрибутов и порядок событий.
- Проверить, что переиспользуемые типы не определены inline без необходимости.

## Related Rules

- Создание сущностей: `rules-ai/frontend/rules-for-creating-new-entity-by-frontend.md`
- Фильтры списков: `rules-ai/frontend/filter-components-guide.md`
