# spec.md — Онлайн-магазин книг (ER-модель)

## Намір
Описати дані книжкового магазину як ER-модель: сутності, атрибути, зв'язки, кардинальності, PK/FK. Без SQL DDL.

## Сутності та атрибути
Усі ідентифікатори — `uuid`. Імена полів — `snake_case`, однакові в spec і в діаграмі.

- **User**: `id` PK, `email` (унікальний), `full_name`, `password_hash` (string), `created_at` (datetime)
- **Publisher**: `id` PK, `name`, `country`
- **Book**: `id` PK, `publisher_id` FK→Publisher, `title`, `isbn` (унікальний), `price` (decimal, поточна ціна), `stock_quantity` (int), `published_year` (int)
- **Author**: `id` PK, `full_name`, `birth_year` (int, необов'язково)
- **Category**: `id` PK, `name` (унікальна)
- **Order**: `id` PK, `user_id` FK→User, `status` (string), `created_at` (datetime)
- **OrderItem** (асоціативна сутність): `id` PK, `order_id` FK→Order, `book_id` FK→Book, `quantity` (int), `unit_price` (decimal, ціна на момент покупки)
- **Review**: `id` PK, `user_id` FK→User, `book_id` FK→Book, `rating` (int 1–5), `comment` (string), `created_at` (datetime)

## Зв'язки та кардинальності
1. Publisher 1 — 0..N Book (видавець має 0+ книг; книга має рівно одного видавця).
2. Book N — M Author: чистий «багато-до-багатьох», **без** сполучної таблиці (книга має 1+ авторів; автор має 0+ книг).
3. Book N — M Category: чистий «багато-до-багатьох» (книга має 0+ категорій).
4. User 1 — 0..N Order (замовлення належить рівно одному користувачу).
5. Order 1 — 1..N OrderItem (замовлення має мінімум одну позицію).
6. Book 1 — 0..N OrderItem (книга може ще не входити в жодне замовлення).
7. User 1 — 0..N Review; Book 1 — 0..N Review. Бізнес-правило: пара (user, book) має щонайбільше один відгук. Review — самостійна сутність (власний `id`, оцінка, текст), а не сполучна.

## Критерії прийняття
- **AC1.** Модель у 3НФ: немає транзитивних залежностей (напр. дані видавця — лише в Publisher).
- **AC2.** Усі `id` і FK мають тип `uuid`; жодних string/number-id.
- **AC3.** Кожна сутність має позначений PK; кожен зовнішній ключ — позначений FK; унікальні поля — UK.
- **AC4.** Немає сполучних таблиць для Book–Author та Book–Category; зв'язки задані напряму як M:N.
- **AC5.** Асоціативна сутність лише одна — OrderItem, бо зв'язок несе власні атрибути (`quantity`, `unit_price`).
- **AC6.** Кардинальності в діаграмі збігаються з розділом «Зв'язки» (включно з опційністю).
- **AC7.** Назви й типи полів у `erd.md` дослівно збігаються зі spec (`price` ≠ `unit_price`: перше — ціна в каталозі, друге — знімок на момент покупки).
- **AC8.** Жодних SQL/ORM-артефактів; діаграма Mermaid успішно рендериться.
