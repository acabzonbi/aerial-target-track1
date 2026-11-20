# Лабораторна 4

### Юнцов Олександр Олександрович ІМ-о51

Обрана форма: Activity діаграма у Mermaid flowchart, де swimlanes реалізовано як subgraph (по одному на учасника).

## UC з лаби 2: Створювати підписки на міста

```gherkin

Функціонал: Створювати підписки на міста
  Як зареєстрований користувач (USER)
  Я хочу створити підписку на місто з радіусом відстеження
  Щоб отримувати повідомлення про повітряні загрози поблизу обраного міста

  Передумова:
    Дано користувач авторизований у системі як USER
    І довідник міст України доступний

  Сценарій: Успішне створення підписки на місто
    Дано користувач відкрив форму створення підписки
    Коли користувач обирає місто "Київ"
    І вводить радіус відстеження 20км
    І підтверджує створення підписки
    Тоді в системі створюється нова підписка
    І підписка містить місто "Київ"
    І підписка містить радіус 20км
    І користувач бачить повідомлення про успішне створення

  Сценарій: Некоректні вхідні дані радіуса
    Дано користувач відкрив форму створення підписки
    Коли користувач обирає місто "Львів"
    І вводить радіус відстеження (хибний радіус)
    І підтверджує створення підписки
    Тоді підписка не створюється
    І користувач бачить повідомлення про помилку

  Сценарій: Місто не знайдено або не вибрано
    Дано користувач відкрив форму створення підписки
    Коли користувач вводить місто (хибне місто)
    І вводить радіус відстеження 30км
    І підтверджує створення підписки
    Тоді підписка не створюється
    І користувач бачить повідомлення про помилку

```


## Ролі

| Роль (swimlane) | Тип | Відповідальність у процесі | Сутність ER |
|---|---|---|---|
| USER | Актор | Обирає місто, вводить радіус, підтверджує | USER |
| SubscriptionForm | UI | Збирає введення, надсилає запит, показує результат | рівень інтерфейсу |
| SubscriptionService | Логіка | Керує процесом, валідує, координує CityCatalog і Repository | рівень логіки |
| CityCatalog | Дані | Пошук міста за назвою | CITY |
| SubscriptionRepository | Дані | Збереження підписки | SUBSCRIPTION |


## Кроки процесу
| # | Роль | Крок | Тип |
|---|---|---|---|
| 1 | USER | openCreateForm() | Початок |
| 2 | SubscriptionForm | renderForm() | Дія |
| 3 | USER | Обирає або вводить місто, вводить радіус (selectCity/enterCity, enterRadius) | Дія |
| 4 | USER | submitSubscription() | Дія |
| 5 | SubscriptionForm - SubscriptionService | createSubscription(userId, city, radius) | message 1 (M1) |
| 6 | SubscriptionService | Приймає M1 і запускає паралельні гілки | Fork |
| 7a | SubscriptionService - CityCatalog | findCityByName(city) | message 2 (M2) branch A |
| 7b | CityCatalog - SubscriptionService | City(id, name) або null | message 3 (M3) branch A |
| 7c | SubscriptionService | Очікує M3, таймаут T1 | Accept/таймаут branch A |
| 8 | SubscriptionService | validateRadius(radius) | Дія branch B |
| 9 | SubscriptionService | Об'єднує гілки | Join |
| 10 | SubscriptionService | Перевіряє результати (D1, D2, D3) | Розгалуження |
| 11 | SubscriptionService - SubscriptionRepository | save(userId, cityId, radiusKm) | message 4 (M4) |
| 12 | SubscriptionRepository - SubscriptionService | Subscription(id) | message 5 (M5) |
| 13 | SubscriptionService | Очікує M5, таймаут T2 | Accept/таймаут |
| 14 | SubscriptionService - SubscriptionForm | SubscriptionCreated(id) | message 6 (M6) |
| 15 | SubscriptionForm - USER | showMessage("Підписку створено"); USER бачить результат | Кінець (успіх) |
| E | SubscriptionService - SubscriptionForm | Помилка (тип залежить від гілки) | message 7 (M7) |
| E | SubscriptionForm - USER | showError(текст за типом помилки); USER бачить результат | Кінець (помилка) |

## Розгалуження

## Повідомлення

## Аудит