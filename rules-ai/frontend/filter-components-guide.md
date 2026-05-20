---
doc_status: current
last_reviewed: 2026-03-24
scope: frontend_filters
when_to_read: create_or_edit_list_filters
max_read_lines: 330
---

# Руководство по переиспользуемым компонентам фильтров

## Must Follow

- Перед созданием нового filter-компонента проверять текущий набор `frontend/components/commons/form/filter/`.
- Предпочитать переиспользование и расширение существующих filter-компонентов.
- Соблюдать проектные Vue-правила из `rules-ai/frontend/rules-for-vue-component.md`.
- При необходимости нового переиспользуемого компонента обновлять `frontend/reusable-components-registry.md`.

## Обзор

Создана система переиспользуемых компонентов фильтров для стандартизации и упрощения разработки списков сущностей в ERP системе.

## Структура компонентов

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

## Компоненты

### 1. CommonsFormFilterText

Базовый текстовый фильтр с возможностью настройки паттерна фильтрации.

```vue
<CommonsFormFilterText
	v-model="filterValue"
	label="Название"
	:dense="true"
	input-class="q-mb-md"
	:filter-pattern="/[^a-zA-Z0-9]/g"
	placeholder="Введите значение"
/>
```

**Props:**
- `modelValue` (String) - значение фильтра
- `label` (String, required) - подпись поля
- `dense` (Boolean, default: true) - компактный режим
- `inputClass` (String, default: 'q-mb-md') - CSS класс
- `filterPattern` (RegExp, optional) - паттерн для фильтрации ввода
- `placeholder` (String, optional) - placeholder

### 2. CommonsFormFilterEmail

Специализированный фильтр для email адресов с автоматической фильтрацией.

```vue
<CommonsFormFilterEmail
	v-model="emailFilter"
	label="Email"
	:dense="true"
/>
```

**Особенности:**
- Автоматически фильтрует ввод (только латинские буквы, цифры, @ и .)
- Блокирует ввод недопустимых символов через клавиатуру
- Фильтрует вставку недопустимых символов
- Предотвращает ввод кириллицы и специальных символов

### 3. CommonsFormFilterInn

Фильтр для ИНН с валидацией только цифр.

```vue
<CommonsFormFilterInn
	v-model="innFilter"
	label="ИНН"
	:dense="true"
/>
```

**Особенности:**
- Принимает только цифры
- Автоматически удаляет все нецифровые символы

### 4. CommonsFormFilterPhone

Фильтр для телефонных номеров.

```vue
<CommonsFormFilterPhone
	v-model="phoneFilter"
	label="Телефон"
	:dense="true"
/>
```

**Особенности:**
- Разрешает цифры, +, -, (, ), пробелы
- Подходит для российских и международных номеров
- Автоматически фильтрует ввод недопустимых символов
- Автоматически очищает от форматирования при отправке на бекенд
- Интегрирован с поиском по связанной таблице `phones`
- Использует `watch` для реактивной фильтрации ввода

### 5. CommonsFormFilterSelect

Выпадающий список для фильтрации.

```vue
<CommonsFormFilterSelect
	v-model="statusFilter"
	label="Статус"
	:options="statusOptions"
	:dense="true"
	:clearable="true"
/>
```

**Props:**
- `modelValue` (String|Number) - выбранное значение
- `label` (String, required) - подпись поля
- `options` (Array, required) - массив опций
- `dense` (Boolean, default: true) - компактный режим
- `inputClass` (String, default: 'q-mb-md') - CSS класс
- `clearable` (Boolean, default: true) - возможность очистки

### 6. CommonsFormFilterDateRange

Фильтр диапазона дат.

```vue
<CommonsFormFilterDateRange
	v-model="dateRange"
	label="Период создания"
	:dense="true"
/>
```

**Особенности:**
- Возвращает объект `{ from: 'YYYY-MM-DD', to: 'YYYY-MM-DD' }`
- Автоматическая валидация дат
- Возможность очистки обеих дат

### 7. CommonsFormFilterNumberRange

Фильтр диапазона чисел.

```vue
<CommonsFormFilterNumberRange
	v-model="amountRange"
	label="Сумма"
	:dense="true"
	step="0.01"
/>
```

**Props:**
- `modelValue` (Object) - объект `{ from: '', to: '' }`
- `label` (String, default: 'Диапазон') - подпись поля
- `dense` (Boolean, default: true) - компактный режим
- `inputClass` (String, default: 'q-mb-md') - CSS класс
- `step` (String|Number, default: '0.01') - шаг для числовых полей

**Особенности:**
- Автоматическая фильтрация ввода (только цифры, точка, минус)
- Поддержка десятичных чисел
- Возвращает объект с полями from и to

## Примеры использования

### Базовый список с фильтрами

```vue
<script setup>

	const filterName = ref('');
	const filterEmail = ref('');
	const filterInn = ref('');
	const filterPhone = ref('');
	const filterStatus = ref('');

	const applyFilters = () => {
		const filters = {};
		
		if (filterName.value?.trim()) filters.name = filterName.value.trim();
		if (filterEmail.value?.trim()) filters.email = filterEmail.value.trim();
		if (filterInn.value?.trim()) filters.inn = filterInn.value.trim();
		if (filterPhone.value?.trim()) filters.phone = filterPhone.value.replace(/[\s()-]/g, '');
		if (filterStatus.value?.trim()) filters.status = filterStatus.value.trim();
		
		emit('applyFilters', filters);
	};

	watch([filterName, filterEmail, filterInn, filterPhone, filterStatus], applyFilters);

</script>

<template>
	<div class="flex row q-gutter-md q-pa-md">
		<div class="col col-3">
			<CommonsFormFilterText
				v-model="filterName"
				label="Название"
			/>
			<CommonsFormFilterEmail
				v-model="filterEmail"
			/>
		</div>
			<div class="col col-3">
				<CommonsFormFilterInn
					v-model="filterInn"
				/>
				<CommonsFormFilterPhone
					v-model="filterPhone"
				/>
			</div>
			<div class="col col-3">
				<CommonsFormFilterSelect
					v-model="filterStatus"
					label="Статус"
					:options="statusOptions"
				/>
			</div>
	</div>
</template>
```

### Сложный фильтр с диапазонами

```vue
<script setup>

	const filterName = ref('');
	const filterDateRange = ref({ from: '', to: '' });
	const filterAmountRange = ref({ from: '', to: '' });

	const applyFilters = () => {
		const filters = {};
		
		if (filterName.value?.trim()) filters.name = filterName.value.trim();
		if (filterDateRange.value.from) filters.created_from = filterDateRange.value.from;
		if (filterDateRange.value.to) filters.created_to = filterDateRange.value.to;
		if (filterAmountRange.value.from) filters.amount_from = filterAmountRange.value.from;
		if (filterAmountRange.value.to) filters.amount_to = filterAmountRange.value.to;
		
		emit('applyFilters', filters);
	};

	watch([filterName, filterDateRange, filterAmountRange], applyFilters, { deep: true });

</script>

<template>
	<div class="flex row q-gutter-md q-pa-md">
		<div class="col col-4">
			<CommonsFormFilterText
				v-model="filterName"
				label="Название"
			/>
			<CommonsFormFilterDateRange
				v-model="filterDateRange"
				label="Период создания"
			/>
		</div>
		<div class="col col-4">
			<CommonsFormFilterNumberRange
				v-model="filterAmountRange"
				label="Сумма"
				step="0.01"
			/>
		</div>
	</div>
</template>
```

## Преимущества

### 1. Консистентность
- Единообразное поведение во всех списках
- Стандартизированная валидация
- Одинаковый UX

### 2. Переиспользование
- DRY принцип
- Быстрая разработка новых списков
- Легкое поддержание

### 3. Валидация
- Встроенная фильтрация ввода
- Предотвращение ошибок
- Улучшенный UX

### 4. Масштабируемость
- Легкое добавление новых типов фильтров
- Расширяемая архитектура
- Стандартизированные паттерны

## Интеграция с существующими компонентами

### Обновление фильтров компаний

```vue
<!-- Было -->
<q-input
	v-model="filterEmail"
	label="Email"
	@input="filterEmailInput"
/>

<!-- Стало -->
<CommonsFormFilterEmail
	v-model="filterEmail"
/>
```

### Создание новых фильтров

1. Создайте новый компонент в `frontend/components/commons/form/filter/`
2. Следуйте паттерну существующих компонентов
3. Добавьте экспорт в `index.js`
4. Используйте в списках сущностей

## Рекомендации

1. **Всегда используйте переиспользуемые компоненты** для стандартных типов фильтров
2. **Создавайте специализированные компоненты** для уникальных случаев
3. **Следуйте единому стилю** во всех списках
4. **Тестируйте валидацию** ввода для каждого типа фильтра
5. **Документируйте новые компоненты** при их создании
