Цель: реализовать модуль управления полиморфными задачами для ERP‑системы на стеке Laravel (бэкенд) + Vue.js (фронтенд) с соблюдением принципов чистой архитектуры.

1. Контекст проекта
Используемая архитектура — чистая архитектура (Clean Architecture).

Стек: Laravel 12+, Vue.js 3 (Composition API), TypeScript, Quasar.

СУБД: MySQL.

Все компоненты должны быть переиспользуемыми и слабосвязанными.

2. Требования к полиморфной модели задач
Сущности, к которым может быть привязана задача: Lead, Customer (Позже список будет расширяться).

Поля таблицы tasks:

id (UUID);

title (string, 255 символов);

description (text, nullable);

status (enum: new, in_progress, on_hold, completed, cancelled);

priority (enum: low, medium, high, urgent);

due_date (datetime, nullable);

assigned_to_id (foreign key к users.id);

created_by_id (foreign key к users.id);

entity_type (string — FQCN модели);

entity_id (UUID — ID сущности);

estimated_hours (float, nullable);

actual_hours (float, nullable);

deleted_at (soft delete).

3. Бэкенд (Laravel)
3.1. Доменный слой

Интерфейс PolymorphicTaskEntity с методом tasks(): MorphMany.

Модели Lead, Customer реализуют этот интерфейс.

Модель Task с полиморфной связью taskable().

3.2. Прикладной слой

DTO для создания и обновления задач.

Сервисы:

TaskCreationService — создание задачи с привязкой к любой сущности;

TaskAssignmentService — назначение исполнителя;

TaskStatusService — изменение статуса с валидацией переходов.

3.3. Инфраструктурный слой

Миграции БД (с индексами для entity_type, entity_id).

Валидаторы для входящих данных.

3.4. API (RESTful)

GET /api/tasks — список с фильтрацией по entity_type, статусу, приоритету, исполнителю;

POST /api/tasks — создание (с указанием entity_type и entity_id);

PUT /api/tasks/{id} — обновление;

PATCH /api/tasks/{id}/status — смена статуса;

GET /api/tasks/context/{entity_type}/{entity_id} — задачи для конкретной сущности;

GET /api/tasks/my — персональные задачи пользователя.

3.5. Дополнительные механизмы

Логирование изменений в task_history (поля: task_id, changed_by, field_name, old_value, new_value).

Уведомления через WebSocket (Laravel WebSockets) и email.

Поиск по названию, описанию, контекстным полям сущности.

4. Фронтенд (Vue.js + TypeScript)
4.1. Компоненты

TaskList — таблица/канбан с фильтрами по типу сущности, статусу, исполнителю.

TaskKanbanBoard — канбан‑доска с drag‑and‑drop.

TaskForm — форма создания/редактирования (динамически подгружает контекстные поля в зависимости от entity_type).

TaskDetail — детальная страница с комментариями, историей, вложениями.

EntitySelector — универсальный селектор сущности (проект, заказ и т. д.).

PriorityBadge, StatusChip — визуализация атрибутов.

4.2. Страницы

/tasks — главный экран (вид переключается: таблица ↔ канбан);

/tasks/create — создание с выбором типа сущности;

/tasks/{id} — детальная страница;

/my-tasks — персональные задачи;

/{entity_type}/{entity_id}/tasks — задачи для конкретной сущности.

4.3. Функциональность

Drag‑and‑drop в канбане (Vuetify Draggable).

Inline‑редактирование в таблице.

Real‑time обновления через WebSocket.

Загрузка вложений (файлы, изображения).

Экспорт в CSV/Excel.

5. Ролевая модель и права доступа
Администратор — полный доступ.

Менеджер — создание, назначение, просмотр всех задач.

Исполнитель — просмотр назначенных, смена статуса, комментарии.

Гость — только просмотр.

Права проверяются на уровне сервисов и контроллеров.

6. Технические требования
Laravel: Eloquent ORM, Policies, Resources, API Resources.

Vue.js: Composition API, Pinia для состояния, Axios для API‑запросов.

TypeScript: строгая типизация для фронтенда.

WebSocket: Laravel WebSockets + Pusher.

Документация API: OpenAPI/Swagger.

Тестирование: Unit‑тесты для сервисов, Feature‑тесты для API.

7. Формат вывода
Сгенерируй полный код для реализации модуля, включая:

Миграции БД (PHP) с индексами для полиморфных полей.

Модели (PHP):

Task с полиморфной связью;

Project, Order и др. с реализацией PolymorphicTaskEntity.

Сервисы (PHP): TaskCreationService, TaskAssignmentService, TaskStatusService.

Репозитории (PHP): интерфейсы и реализации.

Контроллеры API (PHP) с аутентификацией и авторизацией.

API‑маршруты (PHP).

Vue‑компоненты (TypeScript + Composition API):

TaskList.vue, TaskKanbanBoard.vue, TaskForm.vue и т. д.;

типы TypeScript для задач и сущностей.

Конфигурацию WebSocket (Laravel WebSockets).

Примеры запросов Postman/Insomnia для тестирования API (коллекция).

Инструкции по установке и настройке (README.md):

миграция БД;

настройка WebSocket;

запуск фронтенда;

примеры использования API.

8. Критерии качества
Код соответствует принципам чистой архитектуры (разделение на слои, инверсия зависимостей).

Валидация данных на бэкенде и фронтенде.

Обработка ошибок с пользовательскими сообщениями.

Комментарии к сложным участкам кода.

Соответствие стандартам Laravel и Vue.js.