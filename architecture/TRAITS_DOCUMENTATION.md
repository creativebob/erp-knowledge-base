---
doc_status: current
last_reviewed: 2026-03-21
---

# Документация по трейтам для работы с телефонами и email

## Обзор

Созданы два трейта для универсальной работы с телефонами и email адресами в любых моделях Laravel:

- `HasPhones` - для работы с номерами телефонов
- `HasEmails` - для работы с email адресами

## Установка

### 1. Подключение трейтов в модели

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use App\Models\Traits\HasPhones;
use App\Models\Traits\HasEmails;

class YourModel extends Model
{
    use HasPhones, HasEmails;
    
    // остальной код модели
}
```

### 2. Миграции

Убедитесь, что в базе данных созданы таблицы:
- `phones` - для хранения номеров телефонов
- `emails` - для хранения email адресов

## Трейт HasPhones

### Отношения

- `phones()` - получить все номера телефонов
- `primaryPhone()` - получить главный номер телефона

### Геттеры

- `$model->primary_phone` - получить главный номер телефона (строка)
- `$model->additional_phones` - получить дополнительные номера (коллекция)
- `$model->all_phones` - получить все номера как массив строк
- `$model->ordered_phones` - получить номера отсортированные (главный первый)

### Методы

- `$model->hasPhones()` - проверить, есть ли номера телефонов
- `$model->hasPrimaryPhone()` - проверить, есть ли главный номер

### Примеры использования

```php
$company = Company::find(1);

// Проверка наличия телефонов
if ($company->hasPhones()) {
    echo "У компании есть телефоны";
}

// Получение главного номера
$mainPhone = $company->primary_phone; // "+7 (904) 124-58-74"

// Получение всех номеров
$allPhones = $company->all_phones; // ["+7 (904) 124-58-74", "+7 (495) 123-45-67"]

// Получение дополнительных номеров
$additionalPhones = $company->additional_phones; // коллекция без главного

// Получение отсортированных номеров
$orderedPhones = $company->ordered_phones; // главный первый, затем по sort
```

## Трейт HasEmails

### Отношения

- `emails()` - получить все email адреса
- `primaryEmail()` - получить главный email адрес

### Геттеры

- `$model->primary_email` - получить главный email адрес (строка)
- `$model->additional_emails` - получить дополнительные email (коллекция)
- `$model->all_emails` - получить все email как массив строк
- `$model->ordered_emails` - получить email отсортированные (главный первый)

### Методы

- `$model->hasEmails()` - проверить, есть ли email адреса
- `$model->hasPrimaryEmail()` - проверить, есть ли главный email

### Примеры использования

```php
$company = Company::find(1);

// Проверка наличия email
if ($company->hasEmails()) {
    echo "У компании есть email адреса";
}

// Получение главного email
$mainEmail = $company->primary_email; // "creativebob@gmail.com"

// Получение всех email
$allEmails = $company->all_emails; // ["creativebob@gmail.com", "info@company.com"]

// Получение дополнительных email
$additionalEmails = $company->additional_emails; // коллекция без главного

// Получение отсортированных email
$orderedEmails = $company->ordered_emails; // главный первый, затем по sort
```

## API Endpoints

### Телефоны

- `GET /api/commons/phones` - получить телефоны
- `POST /api/commons/phones` - создать/обновить телефоны
- `PUT /api/commons/phones/{id}/primary` - сделать телефон главным
- `DELETE /api/commons/phones/{id}` - удалить телефон

### Email

- `GET /api/commons/emails` - получить email адреса
- `POST /api/commons/emails` - создать/обновить email адреса
- `PUT /api/commons/emails/{id}/primary` - сделать email главным
- `DELETE /api/commons/emails/{id}` - удалить email

## Структура данных

### Телефон

```php
[
    'id' => 1,
    'phoneable_type' => 'company',
    'phoneable_id' => 1,
    'number' => '+7 (904) 124-58-74',
    'is_primary' => true,
    'note' => 'Главный номер',
    'sort' => 1,
    'created_at' => '2025-09-07T05:04:50.000000Z',
    'updated_at' => '2025-09-07T05:04:50.000000Z'
]
```

### Email

```php
[
    'id' => 1,
    'emailable_type' => 'company',
    'emailable_id' => 1,
    'email' => 'creativebob@gmail.com',
    'is_primary' => true,
    'note' => 'Главный email',
    'sort' => 1,
    'created_at' => '2025-09-07T05:04:27.000000Z',
    'updated_at' => '2025-09-07T05:04:27.000000Z'
]
```

## Примеры моделей с трейтами

### Company (уже реализовано)

```php
class Company extends Model 
{
    use HasFactory, Sortable, HasPhones, HasEmails;
    // ...
}
```

### User (уже реализовано)

```php
class User extends Authenticatable
{
    use HasFactory, Notifiable, HasPhones, HasEmails;
    // ...
}
```

### LegalForm (пример)

```php
class LegalForm extends Model
{
    use HasFactory, HasPhones, HasEmails;
    // ...
}
```

## Frontend компоненты

Для работы с трейтами на фронтенде используются:

- `frontend/components/commons/form/phone.vue` - компонент для телефонов
- `frontend/components/commons/form/email.vue` - компонент для email
- `frontend/composables/usePhone.js` - логика для телефонов
- `frontend/composables/useEmail.js` - логика для email

## Преимущества использования трейтов

1. **Переиспользование кода** - один раз написали, используем везде
2. **Единообразие** - одинаковый API для всех моделей
3. **Легкость поддержки** - изменения в одном месте
4. **Гибкость** - можно использовать только нужные трейты
5. **Расширяемость** - легко добавить новые методы в трейты

## Тестирование

Все трейты протестированы и работают корректно:

- ✅ Создание и получение данных
- ✅ Установка главного элемента
- ✅ Получение дополнительных элементов
- ✅ Сортировка элементов
- ✅ Проверка наличия элементов
- ✅ Работа с разными моделями (Company, User)
