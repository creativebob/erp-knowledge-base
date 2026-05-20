---
doc_status: current
last_reviewed: 2026-05-13
---

# Reusable Components Registry

Краткий реестр переиспользуемых UI-компонентов проекта.  
Цель: перед созданием нового компонента сначала проверить, нет ли подходящего в этом списке.

## Как использовать

- Перед созданием нового компонента сначала проверить этот файл и поискать по `frontend/components/commons`.
- Если найден близкий компонент, расширить его через параметры/слоты вместо копирования логики.
- Если создан новый shared-компонент, добавить запись в этот реестр сразу в том же MR/PR.

## Компоненты

- `frontend/components/commons/decor/accessExternalControlCell.vue` (`CommonsDecorAccessExternalControlCell`)
  - Назначение: ячейка «Доступ» для внешнего управления — `CommonsDecorAccessStatusIcon` + кнопка «Отправить ссылку на вход» при праве обновления и допустимом статусе (`canSendExternalLoginLinkStatus`).
  - Когда использовать: `index` вендоров / клиентов / производителей; на странице остаётся одна `PlatformExternalEntryModalSend`, компонент шлёт `openSendLink` с `{ targetId, delivery }`.
  - Пропсы: `status`, `entityAlias` (для `canUpdate`), `recordId` (если `null` — кнопка скрыта), `delivery` из `external_entry_delivery`.

- `frontend/components/platform/external-entry/modal/send.vue` (`PlatformExternalEntryModalSend`)
  - Назначение: модальное окно отправки **ссылки на внешнюю точку входа** (по сайту и каналу доставки) для вендора/клиента/производителя с выданным доступом директора.
  - Когда использовать: родитель подписывается на `openSendLink` из `CommonsDecorAccessExternalControlCell`; вызывает `POST /api/external-login-links/send`.
  - Важные пропсы: `modelValue` (открытие), `targetType` (`vendor` | `customer` | `manufacturer`), `targetId`, `delivery` (`{ sms, email }` из API `external_entry_delivery`).
  - Внутри: `CommonsFormSite` с `skip-unassigned-option`, `CommunicationsFormChannel` (`enabled-only`, `allowed-values` по `delivery`).

- `frontend/components/communications/form/channel.vue` (`CommunicationsFormChannel`)
  - Назначение: выбор канала доставки из `GET /communications/channels`.
  - Важные пропсы: `enabledOnly` (только включённые в настройках), `allowedValues` — опциональный whitelist значений `value` (например `['sms', 'email']`), чтобы сузить список под контекст (наличие телефона/email у получателя).

- `frontend/components/commons/layout/center-ai-chat.vue` (`CommonsLayoutCenterAiChat`)
  - Назначение: drawer-чат с ИИ-ассистентом (выбор агента, треды, отправка, async polling, Reverb).
  - Когда использовать: центральная панель layout (`useCenterPanels`), не дублировать на отдельных страницах.
  - API: `ai/conversations`, `ai/agent-profiles/catalog`; документация — `erp.local/docs/domains/ai/orchestration-and-chat.md`.

- `frontend/components/commons/form/copyText.vue`
  - Назначение: компактное отображение текста с кнопкой копирования.
  - Когда использовать: ссылки, токены, идентификаторы, которые нужно быстро скопировать.
  - Важные пропсы: `text`, `displayText`, `emptyText`, `maxLength`.

- `frontend/components/commons/decor/remainingTime.vue`
  - Назначение: отображение остатка времени между двумя датами.
  - Когда использовать: дедлайны, истечение токена, время до события.
  - Логика вывода: `N дней` или `N часов M минут`, для прошедшего срока `Истекло`.
  - Важные пропсы: `fromTime`, `toTime`, `expiredLabel`.

- `frontend/components/commons/decor/phone.vue` (`CommonsDecorPhone`)
  - Назначение: легкий декоратор отображения телефона в index-таблицах без подключения `usePhone`.
  - Когда использовать: любые `index` страницы, где отображается телефон в ячейке таблицы.
  - Поддерживаемые источники: `primaryPhoneSnapshot`, `phone`, `phones` (приоритет в этом порядке).
  - Формат вывода: `+7 (XXX) XXX-XX-XX`, для пустого значения `Не указан` (или кастом через `emptyText`).

- `frontend/components/commons/decor/money.vue` (`CommonsDecorMoney`)
  - Назначение: единообразный вывод денежной суммы с валютой (локаль `ru-RU`).
  - Когда использовать: ячейки таблиц, итоги, карточки — везде, где нужна сумма + суффикс валюты без дублирования форматирования.
  - Важные пропсы: `value`, `currency` (объект с полями из `MoneyCurrency` в `~/types/commons/money`), `showFractions` (дробная часть), `currencyStyle` (`symbol` | `abbreviation`), `emptyText`.

- `frontend/components/commons/form/filter/sellableType.vue` (`CommonsFormFilterSellableType`)
  - Назначение: переиспользуемый фильтр типа продаваемой сущности.
  - Когда использовать: списки/реестры с полем `sellable_type` (прайс-позиции и аналогичные экраны).
  - Опции: `Все` (`null`), `Услуги` (`service`), `Товары` (`product`).

- `frontend/components/commons/form/filter/status/active.vue` (`CommonsFormFilterStatusActive`)
  - Назначение: трёхпозиционный фильтр «Все / Да / Нет» для булевого признака активности в query (`is_active`): в родителя эмитится `''` для «Все», строки `'true'` / `'false'` для опций (как `CommonsFormFilterStatusManager` / `CommonsFormFilterStatusArchive`).
  - Когда использовать: index-фильтры с `usePagination`, где бэкенд ждёт nullable boolean из query в виде строк `true`/`false`.
  - Пропсы: `modelValue` (строка из родителя), `label` (по умолчанию «Активен»), `dense`.

- `frontend/components/sales/leads/form/tabs/catalogFromPrice.vue` (`SalesLeadsFormTabsCatalogFromPrice`)
  - Назначение: универсальный tab выбора позиций из прайса для формы лида.
  - Когда использовать: вкладки `products/services` в лиде с единым UX выбора из прайса.
  - Внутренние блоки: поиск (`catalogPriceSearch.vue`), дерево (`catalogPriceTree.vue`), переключатель вида (`catalogViewModeToggle.vue`), список (`catalogItemsList.vue`).
  - Режимы: `mode="product"` и `mode="service"`; callback добавления позиции пока заглушка (TODO под заказ).

- `frontend/components/sales/leads/form/order/itemsTable.vue` (`SalesLeadsFormOrderItemsTable`)
  - Назначение: Quasar-native таблица позиций заказа в форме лида с inline-редактированием количества и итогом.
  - Когда использовать: блок «пункты заказа» в лидах и аналогичные сценарии draft-заказа.
  - Колонки: наименование, единица, цена, количество, скидка (placeholder), сумма строки.

- `frontend/components/sales/leads/form/ClientCard.vue` (`SalesLeadsFormClientCard`)
  - Назначение: вкладка «Карточка клиента» в форме лида — переключатель компания/ФЛ, DaData по ИНН или ФИО, `POST /sales/customers/lifecycle-create`, ссылка на клиента после привязки.
  - События: `linked(customerId)` — родитель сохраняет лид и синхронизирует черновик заказа.

- `frontend/components/sales/leads/list/filters.vue` (`SalesLeadsListFilters`)
  - Назначение: панель фильтров списка лидов (контакт, email, телефон, ответственный, «Показать черновики»).
  - Когда использовать: только `pages/sales/leads/index.vue`; ответственный — переиспользование `SalesLeadsFormManagerSelector`.
  - События: `applyFilters` (полный объект фильтров для `usePagination` / `updateFilters`).

- `frontend/components/commons/page/header.vue`
  - Назначение: стандартная шапка index-страниц с заголовком, количеством, фильтрами и поиском.
  - Когда использовать: все списковые страницы сущностей.
  - Важные слоты: `actions`, `search`, `filter`, `breadcrumbs`, `picture`.

- `frontend/components/commons/page/search.vue`
  - Назначение: строка поиска для index-страниц с синхронизацией через emit.
  - Когда использовать: типовой поиск в таблицах/реестрах.
  - События: `search`.

- `frontend/components/commons/page/selectedActions.vue`
  - Назначение: массовые действия по выбранным строкам таблицы (bulk delete и доп. действия).
  - Когда использовать: страницы с `selection="multiple"` в `q-table`.
  - Важные пропсы: `selected`, `entity`, `useRemoteDelete`, `enableBulkTransferActions`.

- `frontend/components/commons/table/fullscreenToggle.vue` (`CommonsTableFullscreenToggle`)
  - Назначение: стандартная кнопка разворота `q-table` на весь экран (как у сотрудников).
  - Когда использовать: во всех index-страницах со `q-table`, в слоте `#top="props"` после `CommonsPageSelectedActions` и `q-space`.
  - Важные пропсы: `inFullscreen`, `toggleFullscreen` (из слота верхней панели QTable).

- `frontend/components/commons/page/dateTimeCell.vue`
  - Назначение: единообразный вывод даты и времени в ячейках index-таблиц.
  - Когда использовать: колонки с `datetime` (`starts_at`, `ends_at`, `created_at`), где время должно быть второй строкой серым цветом.
  - Важные пропсы: `value`.

- `frontend/components/commons/form/saveButton.vue`
  - Назначение: унифицированная кнопка сохранения с состояниями `hasChanges`/`validationErrors`.
  - Когда использовать: create/edit формы вместо локальных вариантов кнопок.
  - Важные пропсы: `hasChanges`, `validationErrors`, `formReady`, `loading`, `mode`.

- `frontend/components/commons/form/site.vue`
  - Назначение: селектор сайта с подгрузкой и валидацией.
  - Когда использовать: формы, где нужна привязка к сайту.
  - Важные пропсы: `modelValue`, `required`, `title`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/time.vue`
  - Назначение: универсальный выбор времени (`HH:mm`) для форм и графиков.
  - Когда использовать: любые поля времени вместо прямого `q-input type=\"time\"`.
  - Важные пропсы: `modelValue`, `label`, `required`, `readonly`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/priceEntryServiceEntrySelect.vue` (`CommonsFormPriceEntryServiceEntrySelect`)
  - Назначение: селектор позиции прайса для услуг (поиск только среди `price_entries` c `sellable_type=service`).
  - Когда использовать: формы, где нужно выбрать услугу из прайса, а не вводить `price_entry_id` вручную.
  - Важные пропсы: `modelValue`, `title`, `required`, `disable`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/company.vue`
  - Назначение: селектор компании с фильтрацией и автоподстановкой.
  - Когда использовать: формы с выбором компании.
  - Важные пропсы: `modelValue`, `required`, `autoSelect`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/direction.vue` (`CommonsFormDirection`)
  - Назначение: селектор направления ERP из справочника `directions` (`GET /commons/directions/list`).
  - Когда использовать: формы с привязкой к предметной области платформы (например AI skills через `direction_id`).
  - Важные пропсы: `modelValue` (`number | null`), `title`, `required`, `defaultAlias` (автовыбор по alias, напр. `commons`).
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/entityAlbums.vue`
  - Назначение: универсальный UI-блок привязки альбома к сущности (single/multi-ready), загрузка фото через token pipeline.
  - Когда использовать: формы доменных сущностей, где нужен attach альбомов без дублирования медиа.
  - Важные пропсы: `modelValue`, `mode`, `readonly`, `nameLabel`, `uploadLabel`.
  - События: `update:modelValue`, `validation-changed`, `uploading-changed`.

- `frontend/components/commons/form/emails.vue`
  - Назначение: список email-адресов (мульти/одиночный режим, primary, валидация).
  - Когда использовать: карточки сотрудников, клиентов, приглашения и другие контактные формы.
  - Важные пропсы: `modelValue`, `required`, `single`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/phones.vue`
  - Назначение: список телефонов (маска, primary, валидация, мульти/одиночный режим).
  - Когда использовать: любые формы с телефонными контактами.
  - Важные пропсы: `modelValue`, `required`, `single`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/invitations/shared/staffingScopeSelector.vue`
  - Назначение: каскадный выбор области вакансий для приглашений (`филиал -> отдел -> должность`).
  - Когда использовать: формы кампаний и одиночных приглашений, где должность обязательна.
  - Важные пропсы: `modelValue` с полями `branch_id`, `department_id`, `position_id`.
  - События: `update:modelValue`, `capacity-changed`, `availability-changed`.

- `frontend/components/marketing/menus/list/filters.vue`
  - Назначение: фильтры списка пунктов меню (название, алиас, отображение).
  - Когда использовать: страница `marketing/sites/.../menus/index`.

- `frontend/components/marketing/menus/form.vue`
  - Назначение: форма create/edit пункта меню (вкладки основное/настройки, выбор родителя через `parentOptions`).
  - Когда использовать: страницы создания и редактирования пункта меню внутри навигации сайта.

- `frontend/components/commons/form/unitSelector.vue` (`CommonsFormUnitSelector`)
  - Назначение: выбор величины (категория единиц) и конкретной единицы измерения.
  - Ограничение списка величин: проп `categoryAliases` — массив алиасов категорий (`count`, `weight`, `volume`, `length`, …), по смыслу как `categoryAliases` у `CommonsFormDigitUnit`; пустой массив — все категории с API.
  - Для каталога товаров: константа `PRODUCT_CATALOG_UNIT_CATEGORY_ALIASES` в `frontend/constants/dictionaries/unitCategories.ts`.
  - Дефолт при пустом `modelValue` и `inline`: пропы `defaultCategoryAlias` + `defaultUnitAlias` (например `count` / `piece` из сидера единиц).

- `frontend/components/commons/form/name.vue` (`CommonsFormName`)
  - `defineExpose({ focus })` — программный фокус (модалки).
  - Проп `autofocus` (опционально) для нативного автофокуса у `q-input`.

- `frontend/components/commons/form/digit.vue` (`CommonsFormDigit`)
  - Назначение: числовой ввод с опциональными кнопками − / + (`control`).

- `frontend/components/commons/form/productCategory.vue` (`CommonsFormProductCategory`)
  - Назначение: выбор категории товара из плоского списка с глубиной (`/sales/catalog/products/categories/list`).
  - Когда использовать: формы и фильтры каталога товаров (аналог `CommonsFormServiceCategory` для услуг).
  - Важные пропсы: `modelValue`, `required`, `title`, `selectOnlyLeaves`, `autoSelect`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/productComponentSelect.vue` (`CommonsFormProductComponentSelect`)
  - Назначение: поиск простого товара для добавления в состав набора (bundle): серверный поиск от 2 символов, исключение уже добавленных id.
  - Когда использовать: диалоги и формы, где выбирается компонент набора каталога.
  - API: `GET /sales/catalog/products/components/search` (`bundle_product_id`, `q`, `exclude_ids`).
  - Важные пропсы: `modelValue` (id компонента), `bundleProductId`, `excludedProductIds`, `title`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/priceEntryServiceSelect.vue` (`CommonsFormPriceEntryServiceSelect`)
  - Назначение: async-выбор услуги для формы позиции прайса без полной предзагрузки большого списка.
  - Когда использовать: формы create/edit прайс-позиций и другие сценарии выбора `service_id` из большого каталога.
  - API: `GET /sales/catalog/services` (`search` от 2 символов, `per_page`), `GET /sales/catalog/services/{id}` для подгрузки текущего значения в edit-режиме.
  - Примечание: без preload списка на mount; запросы только по вводу от 2 символов.
  - Важные пропсы: `modelValue` (id или `null`), `title`, `required`, `disable`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/priceEntryProductSelect.vue` (`CommonsFormPriceEntryProductSelect`)
  - Назначение: async-выбор товара для формы позиции прайса по аналогии с выбором услуги.
  - Когда использовать: формы create/edit позиций в прайсах с режимом `product_only`.
  - API: `GET /sales/catalog/products` (`search` от 2 символов, `per_page`), `GET /sales/catalog/products/{id}` для подгрузки текущего значения в edit-режиме.
  - Примечание: без preload списка на mount; запросы только по вводу от 2 символов.
  - Важные пропсы: `modelValue` (id или `null`), `title`, `required`, `disable`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/form/priceEntrySellableSelect.vue` (`CommonsFormPriceEntrySellableSelect`)
  - Назначение: единый async-селектор «товар или услуга» для смешанного режима прайса.
  - Когда использовать: формы create/edit позиций в прайсах с режимом `mixed`.
  - API: параллельный поиск в `GET /sales/catalog/services` и `GET /sales/catalog/products` по `search` от 2 символов.
  - Важные пропсы: `modelValue`, `sellableType`, `title`, `required`, `disable`.
  - События: `update:modelValue`, `update:sellableType`, `validation-changed`.

- `frontend/components/sales/catalog/product/modal/quickCreate.vue` (`SalesCatalogProductModalQuickCreate`)
  - Быстрое создание товара с сохранением последних выборов в `localStorage` (`useLocalJsonPreference`).
  - Тексты: `frontend/src/locales/ru.ui.json`, доступ через `useUiStrings().t('sales.catalog.product.quickCreate.*')`.

- `frontend/components/commons/manufacturer/form/select.vue` (`CommonsManufacturerFormSelect`)
  - Назначение: выбор производителя из справочника (Commons) с поиском по API.
  - Когда использовать: форма товара, фильтры каталога, другие формы с привязкой к `manufacturer_id`.
  - API: `GET /commons/manufacturers` (`search` от 2 символов, `per_page`), при необходимости `GET /commons/manufacturers/{id}` для подгрузки текущего значения.
  - Важные пропсы: `modelValue` (id или `null`), `label`, `required`, `disable`.
  - События: `update:modelValue`, `validation-changed`.

- `frontend/components/commons/manufacturer/form/manufacturerDataTab.vue` (`CommonsManufacturerFormManufacturerDataTab`)
  - Назначение: вкладка полей справочника производителя (код, описание, флаги отображения/системности; страна и сайт — в карточке компании/субъекта) для страниц создания/редактирования в связке с `BusinessCompanyForm` / `PersonsForm` (слот `customer-data-tab`).
  - Когда использовать: `commons/manufacturers/create`, `commons/manufacturers/[id]` (паттерн как `ProcurementVendorFormVendorDataTab`).
  - Важные пропсы: `modelValue` (`ManufacturerFormData`).
  - События: `update:modelValue`.

- `frontend/components/commons/task/assignee-select.vue` (`CommonsTaskAssigneeSelect`)
  - Назначение: переиспользуемый async-селектор исполнителя задач на базе `q-select` с удалённой фильтрацией.
  - Когда использовать: формы задач и любые формы назначения ответственного сотрудника в рамках tenant/branch-политик.
  - API: `GET /business/managment/employees/select/assignees` с параметрами `q`, `branch_id`, `cross_branch`, `user_id`.
  - Важные пропсы: `modelValue`, `label`, `required`, `disable`, `clearable`, `branchId`, `crossBranch` (по умолчанию `false` = только текущий филиал).
  - События: `update:modelValue`, `validation-changed`, `selected-assignee`.

- `frontend/components/commons/task/modal.vue` (`CommonsTaskModal`)
  - Назначение: общее модальное окно просмотра задачи по `taskId` с быстрыми действиями без перехода на отдельную страницу.
  - Когда использовать: списки/центры задач, где нужен просмотр карточки и операции в контексте текущей страницы.
  - Покрытие: показ полей задачи (название, сущность, дедлайн, статус, описание, исполнители), действия `cancel` (`PATCH /commons/tasks/{id}/status`), `reassign` (`PATCH /commons/tasks/{id}/assignees`), `delete` (`DELETE /commons/tasks/{id}`).
  - Важные пропсы: `modelValue`, `taskId`.
  - События: `update:modelValue`, `updated`, `deleted`.

- `frontend/components/commons/task/create-modal.vue` (`CommonsTaskCreateModal`)
  - Назначение: универсальная модалка создания задачи для любой сущности (`taskable_type` + `taskable_id`).
  - Когда использовать: экраны, где нужно быстро создать связанную задачу без перехода на отдельную страницу.
  - Покрытие: поля `title`, `description`, `due_date`, `assignee_user_ids`; дефолт дедлайна `now + 1 hour`; дефолт исполнителя — текущий пользователь.
  - Важные пропсы: `modelValue`, `taskableType`, `taskableId`, `branchId`.
  - События: `update:modelValue`, `created`.

- `frontend/components/commons/task/form/deadline-presets.vue` (`CommonsTaskFormDeadlinePresets`)
  - Назначение: переиспользуемый блок быстрых пресетов дедлайна для task-форм.
  - Когда использовать: формы и модалки задач рядом с `CommonsFormDate` и `CommonsFormTime`.
  - Покрытие пресетов: `без дедлайна`, `через пару часов`, `завтра`, `к вечеру`, `к концу недели`, `послезавтра`.
  - Важные пропсы: `disabled`.
  - События: `apply-preset` (`none | in_two_hours | tomorrow | evening_today | end_of_week | day_after_tomorrow`).

- `frontend/components/commons/task/entity-list.vue` (`CommonsTaskEntityList`)
  - Назначение: универсальный компактный виджет «Создать задачу + список задач» для конкретной сущности.
  - Когда использовать: карточки/вкладки сущностей (лид, клиент, товар, поставщик), где нужен быстрый task-flow в контексте сущности.
  - Покрытие: список задач по `taskable_type/taskable_id` (`created_at desc`), компактные поля (создана, название, описание, приоритет, статус), действия `Начать/Готово`, обновление через realtime.
  - Важные пропсы: `taskableType`, `taskableId`, `branchId`, `title`, `perPage`.

## Composables (UI)

- `frontend/composables/useUiStrings.js` — строки интерфейса из `src/locales/ru.ui.json` (задел под i18n).
- `frontend/composables/useLocalJsonPreference.js` — возвращает `{ prefs, flush }`: реактивный объект и немедленная запись; `watch` с `deep: true` + `onScopeDispose` (сброс debounce при уходе со страницы).

## Кандидаты на расширение реестра

- `frontend/components/commons/form/filter/select.vue`
- `frontend/components/commons/form/filter/dateRange.vue`
- `frontend/components/commons/form/filter/text.vue`
- `frontend/components/commons/form/email.vue`
