# Spec v2: онлайн-магазин книг (ER)

## Сутності, атрибути та зв'язки
- Customer: `id`, `email`, `full_name`.
  - Створює багато CustomerOrder (1:N). Пише багато Review (1:N).
  - Додає багато Book до списку бажаного (N:M).
- Publisher: `id`, `name`. Випускає багато Book (1:N).
- Author: `id`, `full_name`. Пише багато Book (N:M).
- Category: `id`, `name`. Класифікує багато Book (N:M).
- Book: `id`, `isbn`, `title`, `publication_year`, `price`.
  - Входить у багато CustomerOrder (N:M).
  - Має багато Review (1:N).
- CustomerOrder: `id`, `status`, `created_at`, `total_price` (сума замовлення).
- Review: `id`, `rating`, `comment`, `created_at`.

## Критерії прийняття
1. Усі `id` мають тип `UUID` і обов'язково позначені `PK`.
2. Модель у 3NF.
3. Зв'язки N:M показані прямо між сутностями, без сполучних таблиць.
4. Формат — Mermaid `erDiagram`.
5. У зв'язках 1:N сутність на боці «багато» містить поле зовнішнього ключа з позначкою `FK`.
6. Поля з часом мають тип `timestamp`, не `datetime`.
