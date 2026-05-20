---
doc_status: current
last_reviewed: 2026-04-03
---

# Два типа отклонения верификации приглашения сотрудника

Утверждённый план доработки ядра ERP и клиентов (PWA): разделить **отклонение с возможностью доработки** и **отклонение с отзывом доступа** («не тот человек»). Текущее поведение в коде соответствует в основном **типу A**; тип B — к реализации.

## Контекст (факт из кода)

При обязательной верификации **`Employee` и привязка `User` → `Staff` выполняются только после `approve`**, не при accept. До верификации создаются **`User` + `Person`**, вакантная **`Staff`** по `invitation.scope_id` остаётся свободной. См. `InvitationService::accept`, `finalizeEmployeeInvitation`, `EmployeeLifecycleService::assignUserToStaff`.

## Тип A — возврат на доработку (correction)

**Смысл:** данные неверны / нужны правки; приглашённый исправляет черновик и снова подаёт на проверку.

**Поведение (целевое, частично уже в ядре):**

- Статус приглашения: `rejected` с семантикой «ожидает правок».
- Claim кампании: `rejected`, освобождение слота для повторной подачи по текущим правилам resubmit.
- Черновик `employee_invite_registration`: откат из `ready_for_review` в `draft` (как сейчас после reject).
- Пользователь: остаётся с ограничениями до финального approve (`is_access_blocked` по текущей модели).
- Клиент: после `ready_for_review` — `POST .../resubmit-verification` с `Idempotency-Key`.

**HTTP:** отдельное действие проверяющего (не общий `reject` с телом), например путь вида «reject for correction» / `POST .../reject-for-correction` — имя согласовать при реализации.

## Тип B — отзыв доступа (wrong person / отмена в целом)

**Смысл:** принявший приглашение не должен продолжать сценарий; доступ к тенанту закрыть, продуктовые данные онбординга свернуть.

**Пользователь (мягкий слой, утверждено):**

- Отзыв Sanctum-токенов.
- Сохранение **`accepted_by_user_id` для аудита**; явный признак **«доступ отозван»** в `meta` приглашения и/или состоянии пользователя (в духе уже существующей блокировки).
- Без обязательного физического `DELETE` пользователя по умолчанию.

**Приглашение и данные:**

- Черновик `employee_invite_registration`: **жёстко недоступен** для редактирования (явный статус вроде `voided` / `revoked_by_verifier` + запрет PATCH в политиках/сервисе).
- **Onboarding progress / events:** терминальное состояние «отменено» (новое событие + обновление progress), не silent-delete без следа.
- **Claim кампании:** **`cancelled`** (или эквивалент), не путать с «rejected для доработки» — чтобы воронка не ждала resubmit.

**HTTP:** второе действие проверяющего, отдельный маршрут (например `POST .../revoke-accepted-invitation` / `reject-and-revoke-access`) — имя при реализации.

**Клиент:** resubmit **запрещён**; стабильный **409** с machine-readable кодом при попытке.

## Реализация: принципы

| Принцип | Содержание |
|---------|------------|
| Два маршрута | Проще политики, логи и однозначность «что нажал проверяющий». |
| Аудит | События/attempts с типом действия A vs B; `accepted_by_user_id` не стирать для типа B. |
| Идемпотентность | Повторные POST — безопасны там, где уже принято в client-api (`Idempotency-Key` при необходимости). |
| Согласование с resubmit | `resubmitRejectedEmployeeVerification` разрешать только после reject **типа A** (проверка meta/флага). |

## Готовность к работам

- Продуктовые развилки и границы A/B **согласованы** (этот документ).
- **Реализовано в ядре (2026-04):** `POST /api/invitations/{id}/verify-reject` (тип A, `meta.verification.reject_kind = correction`) и `POST /api/invitations/{id}/verify-revoke-access` (тип B: статус приглашения `revoked`, claim `cancelled`, черновик `voided`, токены принявшего сняты, событие `vendor_educator_verification_access_revoked` → progress `cancelled`). В модалке верификации ERP для `employee_invite` — две кнопки.
- **PWA / client-api:** при `revoked` повторная подача (`resubmit-verification`) запрещена политикой (403); в bootstrap фаза `access_revoked` — `EducatorRegistrationBootstrapResolver`.

## Связанные документы

- Кейс и матрица: [../cases/detstvo/onboarding_educator_task_1_checklist.md](../cases/detstvo/onboarding_educator_task_1_checklist.md)
- PWA, client-api, SMS: [../programs/pwa/external-client-sms-auth.md](../programs/pwa/external-client-sms-auth.md)
- OpenAPI: `erp.local/docs/api/client-v1.openapi.yaml` (onboarding / invitations)
