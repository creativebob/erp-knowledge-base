---
doc_status: current
last_reviewed: 2026-05-20
---

# ER-диаграммы (Obsidian Canvas)

Файлы `*.canvas` — схемы таблиц БД по доменам. Открывать в [Obsidian](https://obsidian.md/) (режим Canvas).

## Как создавать / обновлять

Используйте skill проекта: `.cursor/skills/erp-er-canvas-diagram/` (запрос: «ER-диаграмма для домена ai»).

Правила:

- источник полей — `database/migrations/*_create_*_table.php`;
- файл: `erd/{domain}.canvas`;
- связи — по `foreign()` в миграциях.

## Файлы

| Файл | Домен |
|------|--------|
| [sales.canvas](sales.canvas) | Продажи: лиды, клиенты, заказы, договоры, черновики регистрации |

Черновики общей архитектуры (если есть): `erp.local/docs/tmp/Архитектура ERP*.canvas`.
