---
doc_status: current
last_reviewed: 2026-04-07
---

# ADR: Two-Phase Provisioning для онбординга родителя

## Статус

Accepted

## Контекст

Для `customer`-онбординга проекта Detstvo после верификации заявки нужны как универсальные сущности ERP (клиент/персоны/связи), так и проектные коммерческие эффекты (договор, trial-заказ, лид).

Если выполнять всё в одном обязательном apply ядра, универсальная ERP начинает зависеть от отраслевого сценария Detstvo. Это повышает связанность, усложняет повторное использование и ухудшает устойчивость при частичных сбоях коммерческой части.

## Решение

Вводим двухфазное применение (Two-Phase Provisioning):

1. **D1: Core Apply (универсальный)**
   - Выполняется синхронно после `approve`.
   - Создаёт/обновляет только универсальный контур:
     - `Customer`;
     - `Person` родителя;
     - `Person` ребёнка (по продуктовой схеме);
     - связи с `PersonGroup` и обязательные доменные связи customer-flow.
   - Идемпотентность обязательна (повтор вызова не создаёт дубли).
   - Фиксирует событие фазы `customer_parent_core_applied`.

2. **D2: Program Provisioning (проектный)**
   - Запускается как расширение после успешного D1.
   - Для Detstvo включает:
     - создание/инициацию `Lead`;
     - создание `SaleOrder` c trial-услугой;
     - создание `SalesContract` по правилам домена продаж.
   - Реализуется через существующие сервисы Sales/Subscriptions, без копирования бизнес-логики в invitation/onboarding слой.
   - Идемпотентность обязательна на уровне orchestrator/ключей.
   - Фиксирует событие `customer_parent_program_provisioned`.

## Границы ответственности

- **Ядро ERP (общее):**
  - знает только D1 и факт того, что можно вызвать program-extension;
  - не содержит детсад-специфичных условий.
- **Расширение программы (Detstvo):**
  - реализует D2 и его бизнес-правила;
  - может развиваться независимо от универсального apply.

## Контракт расширения (рекомендация)

Использовать интерфейс расширения по `flow`/program-контексту:

```php
interface RegistrationDraftProgramProvisioningExtension
{
	public function supports(string $flow, int $companyId, ?int $siteId): bool;

	public function provision(ProgramProvisioningContextDTO $context): ProgramProvisioningResultDTO;
}
```

Минимальный `ProgramProvisioningContextDTO`:
- `invitation_id`
- `draft_id`
- `company_id`
- `site_id`
- `customer_id`
- `idempotency_key`
- `actor_user_id`

## Ошибки и поведение при сбоях

- Если D1 не завершился: D2 не запускается.
- Если D1 завершился, а D2 упал:
  - состояние ядра считается валидным;
  - D2 переводится в retriable-состояние (повторяемый запуск);
  - bootstrap может отражать `program_provisioning_status` (`pending|failed|completed`) без блокировки целостности ядра.

## Последствия

Плюсы:
- универсальное ядро не загрязняется проектной коммерцией;
- выше устойчивость к частичным сбоям;
- проще масштабировать подход на другие программы.

Минусы:
- появляется оркестрация между D1 и D2;
- требуется явный мониторинг/ретраи D2.

## Связанные документы

- [`../cases/detstvo/onboarding_customer_implementation_plan.md`](../cases/detstvo/onboarding_customer_implementation_plan.md)
- [`../cases/detstvo/onboarding_customer_task_1_checklist.md`](../cases/detstvo/onboarding_customer_task_1_checklist.md)
- [`./invitation-verification-reject-types.md`](./invitation-verification-reject-types.md)
