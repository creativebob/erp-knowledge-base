---
doc_status: current
last_reviewed: 2026-03-21
---

# Руководство по реализации фильтров для сущностей ERP

## Обзор

Данное руководство описывает полный процесс создания фильтров для сущностей ERP системы, начиная от фронтенда и заканчивая бекендом. Документ основан на опыте реализации фильтров для компаний и содержит все найденные проблемы и их решения.

## Архитектура фильтрации и поиска

### Общая схема работы

```
Frontend Component → Reusable Filter/Search → URL Update → Backend Request → Database Query
```

1. **Frontend Component** - переиспользуемые компоненты фильтров и поиска
2. **URL Update** - обновление URL с параметрами фильтрации и поиска
3. **Backend Request** - передача фильтров и поиска в API
4. **Database Query** - выполнение запроса к базе данных

### Система поиска

#### Архитектура поиска

```
CommonsPageSearch → usePagination → URL Update → Backend Request → Database Query
```

1. **CommonsPageSearch** - универсальный компонент поиска
2. **usePagination** - управление состоянием поиска с debounce
3. **URL Update** - синхронизация поиска с URL
4. **Backend Request** - передача поискового запроса
5. **Database Query** - выполнение поиска по множественным полям

#### Особенности поиска

- **Debounce 500ms** - предотвращает избыточные запросы
- **Синхронизация с URL** - поиск сохраняется в URL параметрах
- **Совместимость с фильтрами** - поиск и фильтры работают одновременно
- **Глобальный поиск** - поиск по множественным полям и связанным таблицам

## Frontend Implementation

### 1. Компонент поиска

#### CommonsPageSearch

**Особенности:**
- Универсальный компонент для поиска по сущностям
- Автоматическая синхронизация с URL параметрами
- Debounce для оптимизации производительности
- Совместимость с Quasar q-input

```vue
<script setup>
  const route = useRoute();
  const searchText = ref(route.query.search || '');

  const emit = defineEmits(['search']);

  const handleSearch = () => {
    emit('search', searchText.value);
  };

  const clearSearch = () => {
    searchText.value = '';
    handleSearch();
  };

  // Инициализируем поиск при монтировании, если есть параметр в URL
  onMounted(() => {
    if (route.query.search) {
      handleSearch();
    }
  });
</script>

<template>
  <q-input
    v-model="searchText"
    label="Поиск"
    dense
    outlined
    @update:model-value="handleSearch"
  >
    <template v-slot:append>
      <q-icon
        v-if="searchText !== '' && searchText !== null" 
        name="close" 
        class="cursor-pointer"
        @click="clearSearch"
      />
      <q-icon name="search"/>
    </template>
  </q-input>
</template>
```

#### Интеграция поиска в страницы

```vue
<template>
  <CommonsPageHeader>
    <template #search>
      <CommonsPageSearch @search="applySearch" />
    </template>
  </CommonsPageHeader>
</template>

<script setup>
  const applySearch = (search) => {
    // Обновляем URL с поиском (debounce уже в usePagination)
    updateSearch(search);
  };

  const { updateSearch } = usePagination(fetchData);
</script>
```

### 2. Переиспользуемые компоненты фильтров

#### Структура компонентов

```
frontend/components/commons/form/filter/
├── text.vue              # Базовый текстовый фильтр
├── email.vue             # Фильтр для email адресов
├── inn.vue               # Фильтр для ИНН
├── phone.vue             # Фильтр для телефонов
├── select.vue            # Выпадающий список
├── dateRange.vue         # Диапазон дат
├── numberRange.vue       # Диапазон чисел
└── index.js              # Экспорт всех компонентов
```

#### Принципы создания компонентов

1. **Composition API** - все компоненты используют `<script setup>`
2. **1 tab отступ** - соблюдение индентации в script секциях
3. **Вертикальное форматирование** - элементы с >2 атрибутами
4. **defineEmits вертикально** - массив элементов отформатирован
5. **Порядок кода** - defineProps, defineEmits, computed, методы
6. **Пустая строка** после `<script setup>`
7. **Без импортов** - Nuxt auto-imports все composables

#### Пример базового компонента

```vue
<script setup>

	const props = defineProps({
		modelValue: {
			type: String,
			default: ''
		},
		label: {
			type: String,
			required: true
		},
		dense: {
			type: Boolean,
			default: true
		},
		inputClass: {
			type: String,
			default: 'q-mb-md'
		}
	});

	const emit = defineEmits([
		'update:modelValue'
	]);

	const filterValue = computed({
		get: () => props.modelValue,
		set: (value) => emit('update:modelValue', value)
	});

</script>

<template>
	<q-input
		v-model="filterValue"
		:label="label"
		:dense="dense"
		clearable
		filled
		outlined
		:class="inputClass"
	/>
</template>
```

### 2. Специализированные компоненты

#### CommonsFormFilterEmail

**Особенности:**
- Блокирует ввод невалидных символов через клавиатуру
- Фильтрует вставку невалидных символов
- Предотвращает ввод кириллицы и специальных символов

```vue
<script setup>

	const props = defineProps({
		modelValue: {
			type: String,
			default: ''
		},
		label: {
			type: String,
			default: 'Email'
		},
		dense: {
			type: Boolean,
			default: true
		},
		inputClass: {
			type: String,
			default: 'q-mb-md'
		}
	});

	const emit = defineEmits([
		'update:modelValue'
	]);

	const emailValue = computed({
		get: () => props.modelValue,
		set: (value) => emit('update:modelValue', value)
	});

	const handleInput = (value) => {
		// Фильтруем только латинские буквы, цифры, @ и .
		const filtered = value.replace(/[^a-zA-Z0-9@.]/g, '');
		if (filtered !== value) {
			emailValue.value = filtered;
		}
	};

	const handlePaste = (event) => {
		// Блокируем вставку невалидных символов
		event.preventDefault();
		const pastedText = (event.clipboardData || window.clipboardData).getData('text');
		const filtered = pastedText.replace(/[^a-zA-Z0-9@.]/g, '');
		emailValue.value = filtered;
	};

	const handleKeydown = (event) => {
		// Блокируем ввод невалидных символов через клавиатуру
		const allowedKeys = [
			'Backspace', 'Delete', 'Tab', 'Enter', 'ArrowLeft', 'ArrowRight', 
			'ArrowUp', 'ArrowDown', 'Home', 'End'
		];
		
		if (allowedKeys.includes(event.key)) {
			return; // Разрешаем служебные клавиши
		}
		
		// Проверяем, является ли символ валидным для email
		const isValidChar = /[a-zA-Z0-9@.]/.test(event.key);
		if (!isValidChar) {
			event.preventDefault();
		}
	};

</script>

<template>
	<q-input
		v-model="emailValue"
		:label="label"
		:dense="dense"
		clearable
		filled
		outlined
		:class="inputClass"
		@input="handleInput"
		@paste="handlePaste"
		@keydown="handleKeydown"
	/>
</template>
```

#### CommonsFormFilterPhone

**Особенности:**
- Разрешает цифры, +, -, (, ), пробелы
- Автоматически фильтрует ввод недопустимых символов
- Использует `watch` для реактивной фильтрации ввода

```vue
<script setup>

	const props = defineProps({
		modelValue: {
			type: String,
			default: ''
		},
		label: {
			type: String,
			default: 'Телефон'
		},
		dense: {
			type: Boolean,
			default: true
		},
		inputClass: {
			type: String,
			default: 'q-mb-md'
		}
	});

	const emit = defineEmits([
		'update:modelValue'
	]);

	const phoneValue = computed({
		get: () => props.modelValue,
		set: (value) => emit('update:modelValue', value)
	});

	const handleInput = (value) => {
		// Фильтруем только цифры, +, -, (, ), пробелы
		const filtered = value.replace(/[^0-9+\-() ]/g, '');
		if (filtered !== value) {
			phoneValue.value = filtered;
		}
	};

	// Отслеживаем изменения значения и фильтруем их
	watch(() => props.modelValue, (newValue) => {
		if (newValue !== undefined && newValue !== null) {
			const filtered = newValue.replace(/[^0-9+\-() ]/g, '');
			if (filtered !== newValue) {
				phoneValue.value = filtered;
			}
		}
	}, { immediate: true });

</script>

<template>
	<q-input
		v-model="phoneValue"
		:label="label"
		:dense="dense"
		clearable
		filled
		outlined
		:class="inputClass"
		@input="handleInput"
	/>
</template>
```

### 3. Компонент фильтров сущности

#### Структура компонента

```vue
<script setup>

	const props = defineProps({
		dense: {
			type: Boolean,
			default: true
		}
	});

	const emit = defineEmits([
		'applyFilters'
	]);

	// Объявляем все фильтры
	const filterName = ref('');
	const filterEmail = ref('');
	const filterPhone = ref('');
	const filterInn = ref('');
	const filterStatus = ref('');

	const applyFilters = () => {
		// Собираем только непустые фильтры
		const filters = {};
		
		if (filterName.value && filterName.value.trim()) {
			filters.name = filterName.value.trim();
		}
		
		if (filterEmail.value && filterEmail.value.trim()) {
			filters.email = filterEmail.value.trim();
		}
		
		if (filterPhone.value && filterPhone.value.trim()) {
			// Очищаем телефон от пробелов, скобок и дефисов для поиска
			filters.phone = filterPhone.value.replace(/[\s()-]/g, '');
		}
		
		if (filterInn.value && filterInn.value.trim()) {
			filters.inn = filterInn.value.trim();
		}
		
		if (filterStatus.value && filterStatus.value.trim()) {
			filters.status = filterStatus.value.trim();
		}
		
		emit('applyFilters', filters);
	};

	// Используем watch для отслеживания изменений в фильтрах
	watch([
		filterName, 
		filterEmail, 
		filterPhone, 
		filterInn, 
		filterStatus
	], () => {
		applyFilters(); // Вызываем applyFilters при изменении любого из фильтров
	});

</script>

<template>
	<q-item-section>
		<div class="flex row q-gutter-md q-pa-md">
			<div class="col col-3 first-wrap">
				<CommonsFormFilterText
					v-model="filterName"
					label="Название"
					:dense="dense"
				/>
				<CommonsFormFilterEmail
					v-model="filterEmail"
					:dense="dense"
				/>
			</div>
			<div class="col col-3 second-wrap">
				<CommonsFormFilterInn
					v-model="filterInn"
					:dense="dense"
				/>
				<CommonsFormFilterPhone
					v-model="filterPhone"
					:dense="dense"
				/>
			</div>
			<div class="col col-3 third-wrap">
				<CommonsFormFilterSelect
					v-model="filterStatus"
					label="Статус"
					:options="statusOptions"
					:dense="dense"
				/>
			</div>
		</div>
	</q-item-section>    
</template>
```

### 4. Интеграция с системой пагинации

#### Обновление usePagination.js

**КРИТИЧЕСКИ ВАЖНО:** Добавить новый фильтр в `filterKeys` в `usePagination.js`:

```javascript
// В usePagination.js строка 34
const filterKeys = [
	'name', 'inn', 'email', 'phone', 'status', 
	'legal_form_id', 'currency_id', 'language_id', 
	'taxation_type_id', 'parent_company_id', 
	'founded_from', 'founded_to', 'created_from', 'created_to'
];
```

#### Новая архитектура usePagination

**Ключевые изменения:**
- **Централизованный debounce** - debounce перенесен в usePagination
- **Универсальная функция updateQuery** - объединяет фильтры и поиск
- **Совместимость фильтров и поиска** - они работают одновременно

```javascript
// Debounce для фильтров и поиска
let filtersTimeout = null;
let searchTimeout = null;

// Функция для обновления фильтров в URL с debounce
const updateFilters = (filters) => {
  if (filtersTimeout) {
    clearTimeout(filtersTimeout);
  }
  
  filtersTimeout = setTimeout(() => {
    updateQuery({ filters, resetPage: true });
  }, 500);
};

// Функция для обновления поиска в URL с debounce
const updateSearch = (search) => {
  if (searchTimeout) {
    clearTimeout(searchTimeout);
  }
  
  searchTimeout = setTimeout(() => {
    updateQuery({ search, resetPage: true });
  }, 500);
};

// Универсальная функция для обновления query параметров
const updateQuery = ({ filters = null, search = null, resetPage = false }) => {
  // Сохраняем текущие параметры пагинации и сортировки
  const newQuery = {
    page: resetPage ? 1 : route.query.page,
    per_page: route.query.per_page,
    sort_by: route.query.sort_by,
    sort_direction: route.query.sort_direction,
    show_filters: route.query.show_filters
  };
  
  // Сохраняем текущий поиск, если он не обновляется
  if (search !== null) {
    newQuery.search = search || undefined;
  } else if (route.query.search) {
    newQuery.search = route.query.search;
  }
  
  // Сохраняем текущие фильтры, если они не обновляются
  const filterKeys = ['name', 'inn', 'email', 'phone', 'status', 'legal_form_id', 'currency_id', 'language_id', 'taxation_type_id', 'parent_company_id', 'founded_from', 'founded_to', 'created_from', 'created_to'];
  
  if (filters !== null) {
    // Обновляем фильтры
    filterKeys.forEach(key => {
      if (filters[key] && filters[key].trim()) {
        newQuery[key] = filters[key].trim();
      }
    });
  } else {
    // Сохраняем существующие фильтры
    filterKeys.forEach(key => {
      if (route.query[key]) {
        newQuery[key] = route.query[key];
      }
    });
  }
  
  // Удаляем undefined значения
  Object.keys(newQuery).forEach(key => {
    if (newQuery[key] === undefined) {
      delete newQuery[key];
    }
  });
  
  router.push({ query: newQuery });
};
```

#### Преимущества новой архитектуры

1. **Централизованное управление** - весь debounce в одном месте
2. **Совместимость** - фильтры и поиск работают одновременно
3. **Производительность** - нет двойного debounce
4. **Простота** - компоненты стали проще
5. **Поддерживаемость** - изменения в одном месте

#### Обновление страницы списка

**КРИТИЧЕСКИ ВАЖНО:** Добавить новый фильтр в `filterKeys` в странице списка:

```javascript
// В pages/business/companies/index.vue строка 114
const filterKeys = [
	'name', 'inn', 'email', 'phone', 'status', 
	'legal_form_id', 'currency_id', 'language_id', 
	'taxation_type_id', 'parent_company_id', 
	'founded_from', 'founded_to', 'created_from', 'created_to'
];
```

## Backend Implementation

### 1. DTO для фильтров

#### Структура DTO

```php
<?php

namespace App\DTO\Business;

use Spatie\LaravelData\Data;
use Spatie\LaravelData\Attributes\Validation;

class CompanyFiltersDTO extends Data
{
	public function __construct(
		#[Validation\StringType]
		#[Validation\Max(255)]
		public readonly ?string $name = null,
		
		#[Validation\StringType]
		#[Validation\Max(12)]
		public readonly ?string $inn = null,
		
		#[Validation\Email]
		#[Validation\Max(255)]
		public readonly ?string $email = null,
		
		#[Validation\StringType]
		#[Validation\Max(20)]
		public readonly ?string $phone = null,
		
		#[Validation\StringType]
		#[Validation\Max(50)]
		public readonly ?string $status = null,
		
		// ... другие фильтры
	) {
		$this->validateBusinessRules();
	}

	// Предметная валидация - проверка бизнес-правил
	public function validateBusinessRules(): void
	{
		// Проверка существования связанных сущностей
		// Проверка логичности дат
		// Другие бизнес-правила
	}
}
```

### 2. Request для валидации

#### Структура Request

```php
<?php

namespace App\Http\Requests\Business;

use App\Http\Requests\Common\IndexRequest;

class CompanyIndexRequest extends IndexRequest
{
	public function rules()
	{
		$rules = parent::rules();
		
		// Техническая валидация для компаний
		$rules = array_merge($rules, [
			// Фильтры - техническая проверка типов
			'filters' => 'nullable|array',
			'filters.name' => 'nullable|string|max:255',
			'filters.inn' => 'nullable|string|max:12',
			'filters.email' => 'nullable|max:40',
			'filters.phone' => 'nullable|string|max:20',
			'filters.status' => 'nullable|string|max:50',
			
			// Поддержка фильтров в корневых параметрах
			'name' => 'nullable|string|max:255',
			'inn' => 'nullable|string|max:12',
			'email' => 'nullable|max:40',
			'phone' => 'nullable|string|max:20',
			'status' => 'nullable|string|max:50',
		]);
		
		return $rules;
	}
}
```

### 3. Service для бизнес-логики

#### Метод applyFilters

```php
private function applyFilters($query, CompanyFiltersDTO $filters)
{
	// Текстовые фильтры
	$query->when($filters->name, function ($query, $name) {
		$query->where('name', 'like', '%' . $name . '%');
	});

	$query->when($filters->inn, function ($query, $inn) {
		$query->where('inn', 'like', '%' . $inn . '%');
	});

	// Фильтр по email - поиск в основной таблице и связанной
	$query->when($filters->email, function ($query, $email) {
		$query->where(function ($subQuery) use ($email) {
			$subQuery->where('email', 'like', '%' . $email . '%')
					 ->orWhereHas('emails', function ($emailQuery) use ($email) {
						 $emailQuery->where('email', 'like', '%' . $email . '%');
					 });
		});
	});

	// Фильтр по телефону - поиск только в связанной таблице
	$query->when($filters->phone, function ($query, $phone) {
		$query->whereHas('phones', function ($phoneQuery) use ($phone) {
			$phoneQuery->where('number', 'like', '%' . $phone . '%');
		});
	});

	// Фильтр по статусу
	$query->when($filters->status, function ($query, $status) {
		$query->where('status', $status);
	});

	return $query;
}
```

#### Метод applySearch

```php
private function applySearch($query, string $search)
{
	return $query->where(function ($query) use ($search) {
		$query->where('name', 'like', '%' . $search . '%')
			  ->orWhere('inn', 'like', '%' . $search . '%')
			  ->orWhere('email', 'like', '%' . $search . '%')
			  ->orWhere('legal_name', 'like', '%' . $search . '%')
			  ->orWhere('short_name', 'like', '%' . $search . '%')
			  ->orWhereHas('emails', function ($emailQuery) use ($search) {
				  $emailQuery->where('email', 'like', '%' . $search . '%');
			  })
			  ->orWhereHas('phones', function ($phoneQuery) use ($search) {
				  $phoneQuery->where('number', 'like', '%' . $search . '%');
			  });
	});
}
```

## Критические ошибки и их решения

### 1. Отсутствие фильтра в filterKeys

**Проблема:** Фильтр не отслеживается системой пагинации

**Решение:** Добавить фильтр в `filterKeys` в двух местах:
- `frontend/composables/usePagination.js`
- `frontend/pages/business/companies/index.vue`

### 2. Конфликт между фильтрами и поиском

**Проблема:** При применении фильтров удаляется поиск и наоборот

**Решение:** Использовать универсальную функцию `updateQuery` в `usePagination`:
- Сохранять все существующие параметры
- Обновлять только нужные параметры
- Поддерживать совместимость фильтров и поиска

### 3. Двойной debounce

**Проблема:** Debounce применяется и в компонентах, и в usePagination

**Решение:** Перенести debounce в `usePagination`:
- Убрать `useTimeout` из компонентов
- Использовать централизованный debounce
- Упростить компоненты

### 4. Неправильная структура базы данных

**Проблема:** Поиск по несуществующей колонке

**Решение:** Проверить структуру таблицы и использовать правильные колонки:
- Для email: `companies.email` + `emails.email`
- Для телефона: только `phones.number`

### 5. Неполная реактивность компонентов

**Проблема:** Компонент не фильтрует ввод при программном изменении

**Решение:** Использовать `watch` с `immediate: true`:

```javascript
watch(() => props.modelValue, (newValue) => {
	if (newValue !== undefined && newValue !== null) {
		const filtered = newValue.replace(/[^0-9+\-() ]/g, '');
		if (filtered !== newValue) {
			phoneValue.value = filtered;
		}
	}
}, { immediate: true });
```

### 6. Неправильная очистка форматирования

**Проблема:** Форматирование не очищается при отправке

**Решение:** Очищать форматирование в `applyFilters`:

```javascript
if (filterPhone.value && filterPhone.value.trim()) {
	// Очищаем телефон от пробелов, скобок и дефисов для поиска
	filters.phone = filterPhone.value.replace(/[\s()-]/g, '');
}
```

## Чек-лист для создания фильтров и поиска

### Frontend

#### Фильтры
- [ ] Создан переиспользуемый компонент фильтра
- [ ] Компонент следует правилам проекта (Composition API, отступы, порядок)
- [ ] Добавлена валидация ввода (если необходимо)
- [ ] Компонент добавлен в `index.js`
- [ ] Фильтр добавлен в компонент фильтров сущности
- [ ] Фильтр добавлен в `watch` массив
- [ ] Фильтр добавлен в `filterKeys` в `usePagination.js`
- [ ] Фильтр добавлен в `filterKeys` в странице списка
- [ ] Добавлена очистка форматирования (если необходимо)

#### Поиск
- [ ] Добавлен `CommonsPageSearch` в `CommonsPageHeader`
- [ ] Реализован `applySearch` в странице списка
- [ ] Используется `updateSearch` из `usePagination`
- [ ] Поиск работает с фильтрами одновременно
- [ ] Поиск синхронизируется с URL

### Backend

#### Фильтры
- [ ] Добавлен фильтр в DTO
- [ ] Добавлена валидация в Request
- [ ] Добавлена логика фильтрации в Service
- [ ] Проверена структура базы данных
- [ ] Протестирована работа фильтра

#### Поиск
- [ ] Добавлен поиск в DTO (IndexDTO)
- [ ] Добавлена логика поиска в Service (applySearch)
- [ ] Поиск работает по множественным полям
- [ ] Поиск работает по связанным таблицам
- [ ] Протестирована работа поиска

## Примеры использования

### Создание фильтра для новой сущности

1. **Создать компонент фильтра** в `frontend/components/commons/form/filter/`
2. **Добавить в index.js** экспорт компонента
3. **Добавить в компонент фильтров** сущности
4. **Обновить filterKeys** в двух местах
5. **Добавить в DTO** новый фильтр
6. **Добавить валидацию** в Request
7. **Реализовать логику** в Service
8. **Протестировать** работу фильтра

### Создание специализированного фильтра

1. **Определить требования** к валидации
2. **Создать компонент** с нужной логикой
3. **Добавить обработчики** событий (keydown, paste, input)
4. **Использовать watch** для реактивности
5. **Протестировать** все сценарии ввода

## Заключение

Данное руководство содержит все необходимые знания для создания фильтров и поиска в ERP системе. Следуя этому руководству, можно избежать всех найденных ошибок и создать надежную систему фильтрации и поиска.

**Ключевые принципы:**
1. **Переиспользование** - создавать универсальные компоненты
2. **Валидация** - блокировать недопустимый ввод
3. **Реактивность** - использовать watch для отслеживания изменений
4. **Интеграция** - добавлять фильтры и поиск во все необходимые места
5. **Совместимость** - фильтры и поиск должны работать одновременно
6. **Централизация** - debounce и управление состоянием в usePagination
7. **Тестирование** - проверять работу на всех уровнях

**Новые возможности:**
- **Универсальный поиск** - поиск по множественным полям и связанным таблицам
- **Совместимость** - фильтры и поиск работают одновременно
- **Оптимизация** - централизованный debounce для лучшей производительности
- **Синхронизация** - все параметры сохраняются в URL
