---
doc_status: current
last_reviewed: 2026-05-14
---

# План: хранение внешних credentials по компании (мульти-тенант)

## Цель

Реализовать **сущность и API** для хранения учётных данных внешних сервисов **в разрезе компании** (`company_id`): в первую очередь **LLM** (OpenAI, Anthropic и т.д. — свои ключи и биллинг у каждого тенанта). Расширяемость на другие интеграции (не только ИИ) заложить в модель **типа/провайдера**.

`.env` остаётся для **платформенных** секретов и dev; **не** для сотен ключей по компаниям.

**Следующий этап:** план [internal-messaging-chat-plan.md](internal-messaging-chat-plan.md) (внутренний чат и профили ИИ-сотрудников) **опирается** на эту сущность через FK `llm_credential_id` в профиле агента и/или fallback «default на провайдера».

## Имя таблицы: `external_credentials`

**Принято короткое имя** вместо `company_external_credentials`. Тенантная граница задаётся **не префиксом таблицы**, а обязательной колонкой **`company_id`** (FK, индекс, комментарий в миграции). В сервисах и Policy — всегда scope по текущей компании, как у остальных `HasCompany`-моделей.

## Принципы

- Жёсткая изоляция по **tenant** на всех слоях (query, сервис, policy, validation): только `company_id` текущего контекста.
- Секреты в БД — **только зашифрованные** (Laravel `encrypted` cast или явный `encrypt()`); в логах и исключениях — **никогда** полный ключ.
- Именование таблицы **не** `accounts`, чтобы не путать с [банковскими счетами](../../app/Models/Business/Bank/BankAccount.php).
- Миграции — **исходные** `*_create_external_credentials_table.php`, без отдельных additive-миграций по правилам репозитория.
- Backend по гайду: [rules-ai/backend/rules-for-creating-new-entity-guide.md](rules-ai/backend/rules-for-creating-new-entity-guide.md) — Controller, Request, DTO, Service, Policy, Resources, сидер `entities`, `PermissionSeeder`, регистрация в [AuthServiceProvider](../../app/Providers/AuthServiceProvider.php); соответствие [VerifyEntityPolicyMatrixCommand](../../app/Console/Commands/Quality/VerifyEntityPolicyMatrixCommand.php).

## Доменная модель (кратко)

Одна строка в `external_credentials` — одно **подключение** к выбранной строке справочника **`external_integration_services`** в разрезе **`company_id`** (и при необходимости **`site_id`**). Тип интеграции задаётся **FK `integration_service_id`**, а не свободной строкой `provider`; детальная схема колонок, секретов и индексов — в разделе **«Схема таблицы и формы»** ниже.

## Схема таблицы и формы (согласовано для первых интеграций)

Цель после этапа C: можно **создать рабочие записи** для выбранных интеграций (в т.ч. **Яндекс Почта**, **DaData**, **SMS.RU** и задел по экосистеме Яндекса) — с валидацией по **типу сервиса** и без утечки секретов в API.

Ориентир по UX: форма из предыдущего проекта пользователя — **«Источник»** (вендор) + **«Сервис»** (конкретная интеграция из справочника) + **«Сайт»** ([Site](../../app/Models/Marketing/Site/Site.php) в разрезе компании), общие поля названия/описания, блок учётных данных и флаги видимости/«системности».

### Справочник: таблица `external_integration_services`

Отдельная таблица (словарь платформы), **не** привязанная к тенанту: описывает доступные **сервисы** для выпадающего списка и правил валидации.

| Колонка | Назначение |
|---------|------------|
| `id` | PK |
| `vendor_code` | Группировка под «Источник» в UI: `yandex`, `dadata`, `sms_ru`, а для ИИ — отдельные вендоры на продукт (`openai`, `anthropic`, `google`, `openclaw`, `nanabanana`, …) — см. ниже **«ИИ / LLM в справочнике»**. |
| `code` | Уникальный машинный ключ строки: `yandex_mail`, `openai_api`, `openclaw`, … |
| `name` | Подпись для UI (в сидере — русские названия как в старом проекте) |
| `sort` | Порядок в списках |
| `requires_site` | Если `true` — при создании credential поле **`site_id` обязательно** и проверяется принадлежность сайта компании; если `false` — ключ на уровне компании (`site_id` = null). Пороговые значения задать в сидере по смыслу (метрика/почта у сайта; DaData и SMS.RU часто — на компанию). |
| `allows_platform_pool` | Если `true` — при резолве учётных данных для **обычного** тенанта без своей валидной строки допускается использование активных credentials **платформенной** компании и при необходимости общего `.env` по тому же fallback, что у платформы (см. [`ExternalCredentialResolver`](../../../app/Services/Settings/ExternalCredentialResolver.php)). В сидере по умолчанию включено для **DaData**, **SMS.RU**, **Groq**, **YandexGPT (облако)**. |
| Без `timestamps` | По договорённости для статичного справочника (как в правилах проекта для словарей); либо с `timestamps`, если нужен аудит — зафиксировать при реализации. |

**Сидер базовых строк** (`updateOrCreate` по `code`): минимум для MVP — интеграции **Яндекс / DaData / SMS.RU**; плюс **задел под экосистему Яндекса** по аналогии со скриншотом: Директ, Подбор слов, Метрика, Маркет, Вебмастер, **Почта Яндекс**, Диск — каждая строка = свой `code` при `vendor_code = yandex`. Для **DaData** и **SMS.RU** — по одной строке на вендор. **Отдельно** — блок **ИИ / LLM** (таблица ниже). Расширение в будущем = новые строки в справочнике + маппинг валидации по `code`.

### ИИ / LLM в справочнике (задел под сидер и чат)

**Идея UX:** в каскаде «Источник» появляются те же принципы, что для Яндекса: один **vendor_code** = один продукт/семейство, внутри — одна или несколько строк `code`, если у вендора разные API (редко; для большинства LLM достаточно **одной строки на вендор**).

**Типовые учётные данные для LLM-строк:** чаще всего достаточно **`api_key`**; опционально **`meta.base_url`** (self‑hosted или кастомный endpoint, в т.ч. для **OpenClaw** и подобных шлюзов); **`api_secret`** — только если конкретный продукт документирует пару client/secret. **`requires_site`:** для ключей уровня компании (чат, бэкенд) — как правило **`false`** (сайт не обязателен); если позже появится сценарий «ключ только для виджета на сайте X» — можно завести отдельную строку с `requires_site = true` или оставить опциональный `site_id` без жёсткого флага.

**Предлагаемые строки для сидера (имена в `name` — русские в UI):**

| `vendor_code` | `code` | Подпись в UI (пример) | Комментарий |
|---------------|--------|-------------------------|-------------|
| `openai` | `openai_api` | OpenAI API | Классический ключ + опционально `meta.base_url` под совместимые прокси. |
| `anthropic` | `anthropic_api` | Anthropic (Claude) | Только ключ API. |
| `google` | `google_gemini_api` | Google AI (Gemini) | API key из Google AI Studio / Cloud. |
| `mistral` | `mistral_api` | Mistral AI | Ключ + при необходимости endpoint в `meta.base_url`. |
| `openclaw` | `openclaw` | OpenClaw | Шлюз/агент как у вас в стеке: **`api_key`** обязателен; **`meta.base_url`** — если инстанс свой или нестандартный хост; второй секрет — только если у OpenClaw в вашей установке реально есть пара ключей → тогда `api_secret`. |
| `nanabanana` | `nanabanana_api` | NanaBanana | Как озвучено в продукте: минимум **`api_key`**; детали пары ключей/URL уточнить по документации сервиса при подключении. |

**Ещё один универсальный вариант «на потом»** (не обязательно в первый сидер): **`groq`** / `groq_api` — быстрый inference, одна строка. Либо **`yandex_gpt`** / `yandex_gpt_api`, если появится оплата ключом Yandex Cloud — отдельная строка с `vendor_code = yandex`, чтобы не смешивать с почтой/метрикой.

**Связь с этапом 2:** профиль агента (`llm_credential_id`) может ссылаться на запись `external_credentials`, у которой выбран любой из `code` выше; логика «какой провайдер/модель» остаётся в профиле агента или в коде резолва по `integration_service.code`.

**API для формы:** отдать список сервисов (сгруппировать по `vendor_code` для каскада «Источник → Сервис») и при необходимости конфиг полей — либо хардкод маппинга `code → набор полей` на бэкенде/фронте MVP, либо позже колонка `field_schema` JSON в справочнике (не блокирует первый релиз).

Порядок миграций: файл `*_create_external_integration_services_table.php` **раньше** `*_create_external_credentials_table.php` из-за FK.

### Хранение секретов

**Выбранный подход:** типовые **зашифрованные** колонки в БД (Laravel cast `encrypted` на модели) + **несекретные** поля для идентификатора и настроек. Расшифрованные значения **не** отдаются в Resource; при редактировании поля секретов пустые, пока пользователь не введёт новое значение (частичное обновление в сервисе).

### Колонки `external_credentials` (миграция создания таблицы)

| Колонка | Назначение |
|---------|------------|
| `company_id` | FK, индекс, обязательный тенант |
| `site_id` | FK на [`sites`](../../app/Models/Marketing/Site/Site.php), nullable; обязательность задаётся флагом `external_integration_services.requires_site` |
| `integration_service_id` | FK на `external_integration_services` — выбранный **сервис** (вместо свободной строки `provider`) |
| `name` | Название подключения в UI |
| `binding_key` | Служебное поле: группировка по паре сервис + сайт (или «глобально» без сайта); в одной группе может быть несколько строк (ротация ключей), инвариант `is_default` среди активных — на уровне сервиса |
| `description` | Текстовое описание (как «Описание» на скрине) |
| `external_identifier` | Несекретный внешний ID / счётчик из кабинета (поле «Идентификатор (ID)») |
| `account_login` | Логин / ящик и т.д. (несекретный, для отображения и подключений) |
| `api_key` | Зашифровано: токен / API ID в зависимости от сервиса |
| `api_secret` | Зашифровано: «Секрет», второй ключ DaData и т.п. |
| `password` | Зашифровано: пароль SMTP, пароли провайдера |
| `meta` | JSON **только несекретного**: `from_name`, `from_email`, `alias`, `public_url` (публичная страница-ссылка), при необходимости SMTP-хост/порт, если не зашиты в коде |
| `is_active` | Вкл/выкл без удаления |
| `is_default` | Как `is_default` в других сущностях: не больше одного default на группу `(company_id, site_id, integration_service_id)` (при `site_id` = null — отдельная группа company-wide) |
| `display` | Как в остальном ERP: «Отображение на сайте» (булево) |
| `is_system` | Как во всём ERP: системная запись; ограничение редактирования/удаления — по policy при необходимости |
| `timestamps` | Без `softDeletes`: удаление записи — **жёсткое** (секреты не храним в «архиве» по продуктовому решению) |

Индексы: `(company_id, integration_service_id)`, `(company_id, site_id)`; индекс `(company_id, binding_key)` **не уникальный** — в одной группе несколько строк; ровно одно `is_default` среди активных в группе — в [`ExternalCredentialService`](../../app/Services/Settings/ExternalCredentialService.php) при save (и тесты). **Policy/валидация:** `site_id` при наличии должен ссылаться на сайт той же `company_id`.

### Маппинг полей старой формы → новая модель

| Старый UI | Куда в ERP |
|-----------|------------|
| Источник (Яндекс …) | Группировка по `external_integration_services.vendor_code` |
| Сервис (Метрика, Почта …) | `integration_service_id` |
| Сайт | `site_id` |
| Имя | `name` |
| Описание | `description` |
| Идентификатор (ID) | `external_identifier` |
| Имя отправителя / Алиас | `meta.from_name`, `meta.alias` (или отдельные колонки позже) |
| Логин | `account_login` |
| API токен | `api_key` |
| Пароль / Секрет / Повтор пароля | `password`, `api_secret`; повтор пароля — только валидация на клиенте/сервере при сохранении, в БД не хранится |
| Публичная страница (ссылка) | `meta.public_url` |
| Отображать на сайте | `display` |
| По умолчанию для сервиса | `is_default` |
| Системная запись | `is_system` |

### Правила заполнения по `code` сервиса (MVP)

Валидация по `external_integration_services.code` (как раньше по `provider`):

| `code` | Обязательные учётные поля | Примечание |
|--------|---------------------------|------------|
| `yandex_mail` | `name`, `account_login` (email), `password`; опционально `meta.from_*` | SMTP Яндекса по умолчанию в коде |
| `dadata` | `name`, **`api_key` и `api_secret` обязательны** | Зафиксировано в C.1. |
| `sms_ru` | `name`, `api_key` (= API ID) | |
| Прочие `yandex_*` | Зависят от API конкретного продукта — по мере внедрения | |
| `openai_api`, `anthropic_api`, `google_gemini_api`, `mistral_api` | `name`, `api_key`; опционально `meta.base_url` | Типовой LLM без SMTP. |
| `openclaw`, `nanabanana_api` | то же; при необходимости `api_secret` по документации | См. таблицу в **«ИИ / LLM в справочнике»**. |

### Поведение формы (frontend)

1. Выбор **источника** (`vendor_code`) → фильтр списка **сервисов** (`integration_service_id`).
2. Выбор **сайта** (`site_id`) — показывать сайты текущей компании; если у выбранного сервиса `requires_site = false`, поле скрыть или сделать необязательным и сохранять `null`.
3. Общие поля: **name**, **description**, **external_identifier**, флаги, **is_default** при необходимости. В рамках компании допускается **несколько** строк на сочетание **сервис + сайт** (или «глобально» без сайта); ровно одно умолчание среди активных в группе — см. сервис нормализации `is_default`.
4. Динамический блок учётных полей по **`code` выбранного сервиса** (аналог старой формы: токен, пароль, секрет, логин).
5. Секреты: маскирование; после сохранения не подставлять plaintext; «повтор пароля» — только для подтверждения ввода.

Бэкенд: правила по `integration_service_id` / подгруженному `code`; при `update` — шифроколонки обновлять только при непустом вводе.

## API (зафиксировано для реализации)

**HTTP-префикс (Laravel `Route::prefix`, без дублирования `/api` в файле):** `settings` — явная связка с UI «Настройки», отдельный блок в [routes/api.php](../../routes/api.php).

**Middleware (как у остального ERP):** `auth:sanctum`, `refine-tenant-context`, `company-external-access`, `branches`.

**Маршруты MVP:**

| Метод | Путь (относительно префикса `settings`) | Назначение |
|-------|----------------------------------------|------------|
| `GET` | `integration-services` | Список строк справочника `external_integration_services` для каскада «Источник → Сервис» (можно кэшировать на клиенте) |
| `GET` | `external-credentials` | Список credentials компании; **без** полных секретов; маска / `secret_present` |
| `POST` | `external-credentials` | Создание (plaintext секреты один раз → encrypted в БД) |
| `GET` | `external-credentials/{id}` | Одна запись (без раскрытия секретов) |
| `PUT`/`PATCH` | `external-credentials/{id}` | Обновление; пустые поля секретов = не менять |
| `DELETE` | `external-credentials/{id}` | **Жёсткое** удаление строки |

Опционально позже: `POST external-credentials/{id}/test` — проверка ключа (таймаут, без логирования PII).

**Имена ресурсов** в коде (`ExternalCredentialController`, …) привести к этим путям.

### Права (RBAC) — решение C.1

- Сущность **`external_credentials`** в [EntitiesTableSeeder](../../database/seeders/Commons/EntitiesTableSeeder.php): полный CRUD по permission, **Policy** с `$user->hasPermissionTo(...)`; модель зарегистрирована в [AuthServiceProvider](../../app/Providers/AuthServiceProvider.php).
- **По умолчанию создавать/редактировать/удалять** могут пользователи с ролью **`admin`** (и **`owner`** через `syncPermissions(Permission::all())` после [PermissionSeeder](../../database/seeders/Users/PermissionSeeder.php)). Роли **`manager`** / **`hr`** **не** получают create/update/delete без явного расширения сидера.
- Отдельное право **`share external services`** (в [OwnerSeeder](../../database/seeders/Users/OwnerSeeder.php) в блоке `$ownerPermissions`, `action_id` = null): доступ к UI **«Платформа → Общие ключи (пул)»** (`/platform/shared-external-credentials`); в стандартном порядке сидов у **admin** это право не назначается (синхронизация admin до появления записи в `permissions`).

### Хаб фронта `/settings`

Пункт меню и доступ к **обзорной** странице `/settings` (хаб карточек направлений): показывать при **`canView('external_credentials')`** той же сущности (отдельной сущности `system_settings` **не** вводим на MVP).

## Границы

- **Не** домен `erp.local/docs/domains/communications/communications-foundation.md` (`communications_messages`) — это исходящие SMS/email, другой продуктовый смысл.
- **Не** хранить ключи в профиле ИИ-сотрудника — только ссылка на запись credentials (реализуется во втором плане).

## Риски и открытые решения (до старта реализации желательно зафиксировать)

- **Шифрование Laravel (`encrypted`)** завязано на `APP_KEY`: смена ключа приложения без корректной ротации делает старые секреты нечитаемыми — для enterprise-клиентов в будущем можно спланировать KMS/отдельный ключа шифрования; на MVP достаточно осознанного бэкапа ключей и процедуры ротации.
- **Дублирование логики валидации:** маппинг `code → обязательные поля` в PHP Request + зеркально на фронте — риск расхождения; держать **единый** источник правды (класс/конфиг на бэкенде + ответ API «какие поля нужны для выбранного `integration_service_id`») либо хотя бы тесты на матрицу правил.
- **Справочник в БД vs код:** новый `code` в сидере требует деплоя; для редких динамических интеграций позже можно добавить админку справочника — не блокирует MVP.
- **RBAC и хаб `/settings`:** закрыто в C.1 — см. раздел **«API»** и подраздел **«Права (RBAC)»**.
- **Производительность:** объём таблицы credentials обычно мал; при большом числе LLM-запросов узким местом будет вызов API, а не эта сущность.

## Трек исполнения

Статусы в чеклистах: `todo` | `in_progress` | `done` | `blocked`.

### Этап C.1 — Проектирование и согласования

- [x] (`done`) Схема данных: справочник **`external_integration_services`** (вендор + `code` сервиса, сидер строк в т.ч. по скринам Яндекса), **`external_credentials`** с FK на сервис и на **`site_id`**, типовые encrypted-поля + маппинг полей старой формы — см. **«Схема таблицы и формы»**.
- [x] (`done`) Матрица прав: create/update/delete по умолчанию у **`admin`** (и owner); **не** выдавать manager/hr без явного добавления permissions; хаб `/settings` при **`canView('external_credentials')`**.
- [x] (`done`) API: префикс **`settings/`**, таблица путей и жёсткий `DELETE` — см. **«API»**; Resource **без** раскрытия секретов; DaData: **`api_key` + `api_secret` обязательны**.

### Этап C.2 — Backend

- [x] (`done`) Миграция `*_create_external_integration_services_table.php` + модель + [сидер](../../database/seeders/Settings/ExternalIntegrationServicesTableSeeder.php) (`updateOrCreate` по `code`: Яндекс-набор, DaData, SMS.RU, LLM-строки из раздела **«ИИ / LLM»**, Groq, YandexGPT и др.); колонка **`allows_platform_pool`** и пул в [`ExternalCredentialResolver`](../../app/Services/Settings/ExternalCredentialResolver.php).
- [x] (`done`) Миграция `*_create_external_credentials_table.php` + модель `ExternalCredential` + связи с `Site`, `ExternalIntegrationService`.
- [x] (`done`) DTO, `ExternalCredentialService`, Policy, Requests, Controller, V1 Resources; справочник сервисов — `ExternalIntegrationServiceController` + Resource (в т.ч. `allows_platform_pool`).
- [x] (`done`) Роуты в [routes/api.php](../../routes/api.php); локализация `lang/ru/business.php` и связанные ключи для доменных ошибок.
- [x] (`done`) Сущность в [EntitiesTableSeeder](../../database/seeders/Commons/EntitiesTableSeeder.php) + [PermissionSeeder](../../database/seeders/Users/PermissionSeeder.php); в [OwnerSeeder](../../database/seeders/Users/OwnerSeeder.php) — спец-права (`change user`, …) и **`share external services`** для пула.
- [x] (`done`) `php artisan entity-access:verify-matrix` — зелёный (модель с политикой есть в `entities`).

**Опционально позже (не блокирует MVP):** `POST external-credentials/{id}/test`; единый API «схема полей по `integration_service_id`» вместо дублирования правил фронт/бэк.

### Этап C.3 — Frontend (эталонная консистентность)

**Обязательная цепочка правил (читать и соблюдать все три):**

1. [rules-for-vue-component.md](rules-ai/frontend/rules-for-vue-component.md) — автоимпорт composables, порядок секций в `<script setup>`, вертикальные атрибуты, `then/catch/finally`, типы в `frontend/types`.
2. [rules-for-creating-new-entity-by-frontend.md](rules-ai/frontend/rules-for-creating-new-entity-by-frontend.md) — доменная структура файлов, эталон **плоской** сущности `brand` (`pages` + `components` + `types` + `defaults`), `useNotify`, переиспользование общих form/list/saveButton где уместно, реестр компонентов.
3. [filter-components-guide.md](rules-ai/frontend/filter-components-guide.md) — на `index` списка credentials фильтры только через переиспользуемые `frontend/components/commons/form/filter/*`, без одноразовых костылей.

Дополнительные ориентиры по UX настроек: [communications/channel-settings/index.vue](../frontend/pages/communications/channel-settings/index.vue); каркас index/create/edit — как у выбранного эталона из п.2.

**Навигация:** в [frontend/layouts/default.vue](../frontend/layouts/default.vue) в раздел **«Настройки»** добавить пункт на **обзорную** страницу системных настроек (см. ниже) и при необходимости отдельный пункт на список credentials.

**Маршруты (черновик):**

| Путь | Назначение |
|------|------------|
| `/settings/index.vue` | **Перечень** направлений настроек: карточки/список ссылок на подразделы. Первая секция: «Внешние подключения» (LLM и далее другие интеграции) → ведёт на дочерний список. Зарезервировать место под будущие блоки без ломки URL. |
| `/settings/external-credentials/index.vue` | Таблица записей `external_credentials` (источник/сервис из справочника, сайт, метка, активен, default, маска секрета), действия create/edit/delete по правам. |
| при необходимости | `create.vue`, `[id]/edit.vue` — по тому же каркасу, что у других сущностей с отдельными страницами форм. |

**Имя папок компонентов:** по правилам проекта — например `frontend/components/settings/external-credentials/list/filters.vue`, форма — `.../form.vue`; в шаблонах PascalCase (`SettingsExternalCredentialsListFilters`).

**Обязательно:** `definePageMeta` + `middleware: ['auth', 'permissions']` + `permission`/`permissions` как принято в проекте; HTTP через `$axios` с `then/catch/finally`; уведомления через **useNotify** (не `$q.notify`), см. п.2 выше.

**RBAC на фронте:** сущность `external_credentials` в сидере; **хаб** `/settings` — при **`canView('external_credentials')`**; страница пула платформы — при **`share external services`** (см. C.1 и `erp.local/docs/domains/settings/external-credentials.md`).

- [x] (`done`) [`frontend/pages/settings/index.vue`](../frontend/pages/settings/index.vue) — хаб настроек.
- [x] (`done`) [`frontend/pages/settings/external-credentials/`](../frontend/pages/settings/external-credentials/) — index, create, `[id]` (редактирование).
- [x] (`done`) [`frontend/components/settings/external-credentials/`](../frontend/components/settings/external-credentials/) — форма, фильтры списка; типы в [`frontend/types/settings/externalCredential.ts`](../frontend/types/settings/externalCredential.ts).
- [x] (`done`) [default.vue](../frontend/layouts/default.vue) — `routeAliasMap`, пункты «Настройки» и «Платформа → Общие ключи (пул)».
- [x] (`done`) Поля секретов: в форме `type="password"`, после сохранения не подставляются plaintext (логика загрузки/отправки в форме).
- [ ] (`todo`) Запись в [reusable-components-registry.md](frontend/reusable-components-registry.md) — **не делалась** (нет нового вынесенного в общий каталог компонента сверх текущих `list/filters` + формы; добавить при ревью/рефакторинге).

### Этап C.4 — Тесты и приёмка

- [x] (`done`) Feature: изоляция по компании — [`ExternalCredentialApiTest::test_index_does_not_leak_foreign_company_credentials`](../../tests/Feature/Settings/ExternalCredentialApiTest.php).
- [x] (`done`) Feature: **несколько активных строк** в одной группе `(company_id, integration_service_id, site_id)` + нормализация **`is_default`** (в т.ч. перенос дефолта при update, промоушн при delete) — тесты в том же `ExternalCredentialApiTest`; привязка **чужого `site_id`** отклоняется — `test_store_rejects_site_from_other_company`.
- [x] (`done`) Unit: резолв credentials, пул платформы, перебор строк с пустыми секретами — [`ExternalCredentialResolverTest`](../../tests/Unit/Settings/ExternalCredentialResolverTest.php); DaData-маршруты — [`DaDataApiAuthTest`](../../tests/Feature/Api/DaDataApiAuthTest.php); фильтр индекса **`platform_pool_service_only`** — `ExternalCredentialApiTest::test_index_platform_pool_service_only_returns_only_poolable_integrations`.
- [x] (`done`) Чеклист плана и `last_reviewed` — актуализированы.

## Связанные документы

- `erp.local/docs/domains/settings/external-credentials.md` — доменная документация (резолв, пул, env, права).
- [rules-for-vue-component.md](rules-ai/frontend/rules-for-vue-component.md) · [rules-for-creating-new-entity-by-frontend.md](rules-ai/frontend/rules-for-creating-new-entity-by-frontend.md) · [filter-components-guide.md](rules-ai/frontend/filter-components-guide.md) — фронтенд этой фичи.
- [internal-messaging-chat-plan.md](internal-messaging-chat-plan.md) — этап 2, внутренний чат и ИИ.
- Сводный технический контекст ранее вёлся в Cursor-плане `erp_глобальный_чат_ии` (проектные заметки; источник истины для репозитория — эти файлы в `docs/`).
- Локальные файлы `.cursor/plans/*.plan.md` (вне git) — **не** источник истины; при расхождении с кодом см. раздел **«Черновые планы Cursor и источник истины»** в `erp.local/docs/domains/settings/external-credentials.md`.
