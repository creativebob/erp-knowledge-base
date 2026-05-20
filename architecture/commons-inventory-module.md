# Модуль Commons / Inventory (номенклатура склада и производства)

Объединяет сущности уровня **материал, сырьё, упаковка, расходные материалы** — общий домен «то, что учитывается в количественно-весовом выражении», без привязки HTTP-слоя к Sales.

## Принцип

- **Маршруты** могут оставаться под префиксом, удобным для UI (например `sales/catalog/materials`), если экран живёт в разделе продаж.
- **Код** (контроллеры и далее по желанию сервисы/DTO) — в неймспейсе домена, чтобы не раздувать `Commons/` и `Sales/Catalog/` десятками файлов.

## Рекомендуемая раскладка (по мере роста)

| Слой | Путь | Пример |
|------|------|--------|
| HTTP | `app/Http/Controllers/Commons/Inventory/` | `MaterialController.php`, позже `RawMaterialController.php`, `PackagingController.php`, `ConsumableController.php` |
| FormRequest | `app/Http/Requests/Commons/Inventory/` | переносить при добавлении новых сущностей или рефакторинге |
| API Resource | `app/Http/Resources/V1/Commons/Inventory/` | при разрастании `Resources/V1/Commons/` |
| Сервисы | `app/Services/Commons/Inventory/` | когда появятся отдельные сервисы (сейчас `MaterialService` в `Commons/`) |
| DTO | `app/DTO/Commons/Inventory/` | аналогично |
| Модели | `app/Models/Commons/Inventory/` | при выделении поддомена; до рефакторинга допустима плоская `Models/Commons/Material` |

## Текущее состояние

- `MaterialController` → `App\Http\Controllers\Commons\Inventory\MaterialController`
- Остальные классы материала (модель, сервис, DTO, requests в корне `Commons`) можно постепенно переносить в подпапку `Inventory/`, когда появятся соседние сущности.
