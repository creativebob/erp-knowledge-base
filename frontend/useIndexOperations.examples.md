---
doc_status: current
last_reviewed: 2026-03-21
---

# Примеры использования useIndexOperations

Примеры использования composable для различных сущностей на индексных страницах.

// ===========================================
// Пример 1: Страница компаний (companies/index.vue)
// ===========================================
/*
<script setup>
	const { 
		createBulkDeleteHandler, 
		createRecordDeleteHandler,
		createRecordUpdateHandler,
		createRecordCreateHandler 
	} = useIndexOperations();
	
	const rows = ref([]);
	const selected = ref([]);
	const totalRows = ref(0);
	const pagination = ref({});
	
	// Обработчики для разных операций
	const handleBulkDeleted = createBulkDeleteHandler({
		rows,
		totalRows,
		pagination,
		selected
	});
	
	const handleRecordDeleted = createRecordDeleteHandler({
		rows,
		totalRows,
		pagination
	});
	
	const handleRecordUpdated = createRecordUpdateHandler({
		rows
	});
	
	const handleRecordCreated = createRecordCreateHandler({
		rows,
		totalRows,
		pagination
	});
</script>

<template>
	<CommonsPageSelectedActions 
		:selected="selected" 
		entity-endpoint="companies"
		@bulkDeleted="handleBulkDeleted"
	/>
	
	<CommonsPageGroupButtons
		:entity-endpoint="pageMeta?.entity?.endpoint"
		:record-id="props.row.id"
		:record-name="props.row.name"
		@deleted="handleRecordDeleted"
		@updated="handleRecordUpdated"
		@created="handleRecordCreated"
	/>
</template>
*/

// ===========================================
// Пример 2: Страница продуктов (products/index.vue)
// ===========================================
/*
<script setup>
	const { createBulkDeleteHandler, createBulkUpdateHandler } = useBulkOperations();
	
	const rows = ref([]);
	const selected = ref([]);
	const totalRows = ref(0);
	const pagination = ref({});
	
	// Обработчик массового удаления
	const handleBulkDeleted = createBulkDeleteHandler({
		rows,
		totalRows,
		pagination,
		selected
	});
	
	// Обработчик массового обновления (например, изменение статуса)
	const handleBulkUpdated = createBulkUpdateHandler({
		rows,
		selected
	});
</script>

<template>
	<CommonsPageSelectedActions 
		:selected="selected" 
		entity-endpoint="products"
		@bulkDeleted="handleBulkDeleted"
		@bulkUpdated="handleBulkUpdated"
	/>
</template>
*/

// ===========================================
// Пример 3: Страница заказов (orders/index.vue)
// ===========================================
/*
<script setup>
	const { 
		createBulkDeleteHandler, 
		createBulkUpdateHandler,
		createBulkArchiveHandler 
	} = useIndexOperations();
	
	const rows = ref([]);
	const selected = ref([]);
	const totalRows = ref(0);
	const pagination = ref({});
	
	// Обработчики для разных операций
	const handleBulkDeleted = createBulkDeleteHandler({
		rows,
		totalRows,
		pagination,
		selected
	});
	
	const handleBulkArchived = createBulkArchiveHandler({
		rows,
		totalRows,
		pagination,
		selected
	});
</script>

<template>
	<CommonsPageSelectedActions 
		:selected="selected" 
		entity-endpoint="orders"
		@bulkDeleted="handleBulkDeleted"
		@bulkArchived="handleBulkArchived"
	/>
</template>
*/

// ===========================================
// Расширение composable для новых операций
// ===========================================
/*
// Если нужно добавить новую операцию, расширьте useIndexOperations:

export const useIndexOperations = () => {
	// ... существующие методы ...
	
	/**
	 * Обработчик массового экспорта
	 */
	const createBulkExportHandler = ({ selected }) => {
		return (result) => {
			// Логика экспорта
			console.log('Экспортировано:', result.exported);
			selected.value = [];
		};
	};
	
	return {
		// ... существующие методы ...
		createBulkExportHandler
	};
};
*/
