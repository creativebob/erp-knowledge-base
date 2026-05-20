---
doc_status: current
last_reviewed: 2026-03-21
---

# Детальный поток выполнения метода Index

## Пример запроса
```
GET /api/business/companies?page=1&per_page=30&sort_by=name&sort_direction=desc
```

## Пошаговое выполнение

### Шаг 1: Controller
```php
CompanyController::index(CompanyIndexRequest $request)
```
**Входные данные:**
- `page=1`
- `per_page=30` 
- `sort_by=name`
- `sort_direction=desc`

**Действия:**
1. ✅ Валидация через `CompanyIndexRequest`
2. ✅ Создание DTO через `IndexDTO::from()` и `CompanyFiltersDTO::from()`
3. ✅ Вызов сервиса `$this->companyService->index($index, $filters)`
4. ✅ Возврат JSON ответа

---

### Шаг 2: DTO Creation
```php
// В контроллере создаются отдельные DTO
$index = IndexDTO::from($request->validated());
$filters = CompanyFiltersDTO::from($request->validated());
```
**Результат:**
```php
IndexDTO {
    page: 1,
    per_page: 30,
    sort_by: "name",
    sort_direction: "desc",
    search: null
}

CompanyFiltersDTO {
    // фильтры если есть
}
```

---

### Шаг 3: Service Layer
```php
CompanyService::index(IndexDTO $index, ?CompanyFiltersDTO $filters = null)
```

#### 3.1 Базовый запрос
```php
$query = Company::with(['legalForm']);
// SQL: SELECT * FROM companies LEFT JOIN legal_forms ON ...
```

#### 3.2 Применение фильтров
```php
if ($filters) {
    $query = $this->applyFilters($query, $filters);
}
// SQL: WHERE name LIKE '%filter%' AND inn LIKE '%filter%'
```

#### 3.3 Применение поиска
```php
if ($index->search) {
    $query = $this->applySearch($query, $index->search);
}
// SQL: WHERE (name LIKE '%search%' OR inn LIKE '%search%')
```

#### 3.4 Пагинация и сортировка
```php
$companies = $this->applyPaginationAndSortingFromDTO($query, $index);
```

---

### Шаг 4: Trait - Pagination & Sorting
```php
HasDTOPaginationAndSorting::applyPaginationAndSortingFromDTO()
```

#### 4.1 Извлечение параметров
```php
$perPage = 30;        // из IndexDTO->per_page
$page = 1;            // из IndexDTO->page  
$sortBy = "name";     // из IndexDTO->sort_by
$descending = true;   // из IndexDTO->sort_direction === 'desc'
```

#### 4.2 Применение сортировки
```php
if ($sortBy) {
    $query = $query->applySorting($sortBy, $descending ? 'true' : 'false');
}
```

---

### Шаг 5: Model - Sortable Trait
```php
Sortable::applySorting($query, "name", "true")
```

#### 5.1 Проверка и применение
```php
if ("name") {
    $query->orderBy("name", "true" === 'true' ? 'desc' : 'asc');
}
// SQL: ORDER BY name DESC
```

#### 5.2 Пагинация
```php
return $query->paginate(30, ['*'], 'page', 1);
// SQL: LIMIT 30 OFFSET 0
```

**Итоговый SQL:**
```sql
SELECT companies.*, legal_forms.* 
FROM companies 
LEFT JOIN legal_forms ON companies.legal_form_id = legal_forms.id
WHERE (фильтры если есть)
ORDER BY name DESC
LIMIT 30 OFFSET 0
```

---

### Шаг 6: Meta Information
```php
HasDTOPaginationAndSorting::buildMetaInformation()
```

#### 6.1 Формирование мета-данных
```php
$meta = [
    'pagination' => [
        'current_page' => 1,
        'per_page' => 30,
        'total' => 150,        // общее количество записей
        'last_page' => 5,      // последняя страница
        'from' => 1,           // первая запись на странице
        'to' => 30             // последняя запись на странице
    ],
    'sorting' => [
        'sort_by' => 'name',
        'sort_direction' => 'desc'
    ],
    'filters' => [],
    'search' => null,
    'entity' => [
        'title' => 'Компании',
        'description' => 'Список компаний',
        'is_rightable' => true,
        'view_path' => '/business/companies'
    ]
];
```

---

### Шаг 7: Response Formation
```php
return [
    'data' => $companies->items(),  // массив компаний
    'meta' => $meta                 // мета-информация
];
```

---

### Шаг 8: JSON Response
```php
response()->json($result)
```

**Итоговый JSON:**
```json
{
    "data": [
        {
            "id": 1,
            "name": "Компания Z",
            "inn": "1234567890",
            "legal_form": {
                "id": 1,
                "name": "ООО"
            }
        },
        {
            "id": 2, 
            "name": "Компания Y",
            "inn": "0987654321",
            "legal_form": {
                "id": 2,
                "name": "ИП"
            }
        }
        // ... еще 28 записей
    ],
    "meta": {
        "pagination": {
            "current_page": 1,
            "per_page": 30,
            "total": 150,
            "last_page": 5,
            "from": 1,
            "to": 30
        },
        "sorting": {
            "sort_by": "name",
            "sort_direction": "desc"
        },
        "filters": [],
        "search": null,
        "entity": {
            "title": "Компании",
            "description": "Список компаний",
            "is_rightable": true,
            "view_path": "/business/companies"
        }
    }
}
```

## Временная последовательность

```
0ms    ┌─ Request received
1ms    ├─ Validation (CompanyIndexRequest)
2ms    ├─ DTO creation (CompanyIndexDTO)
3ms    ├─ Service call (CompanyService::index)
4ms    ├─ Query building (Company::with)
5ms    ├─ Filters application
6ms    ├─ Search application  
7ms    ├─ Sorting application (Sortable trait)
8ms    ├─ Pagination (paginate)
9ms    ├─ Database query execution
15ms   ├─ Meta information building
16ms   ├─ Response formation
17ms   └─ JSON response sent
```

## Ключевые точки контроля

1. **Валидация** - проверка входных параметров
2. **DTO** - структурирование данных
3. **Query Building** - построение SQL запроса
4. **Sorting** - применение сортировки через трейт
5. **Pagination** - разбиение на страницы
6. **Meta** - формирование мета-информации
7. **Response** - финальный JSON ответ
