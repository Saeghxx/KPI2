# Prompt 2 — EARS-вимоги

## Промпт
Ось актори: [таблиця акторів з кроку 1]. Ось ER-модель: erDiagram
    Customer {
        UUID id PK
        varchar email UK
        varchar full_name
    }

    Publisher {
        UUID id PK
        varchar name
    }

    Author {
        UUID id PK
        varchar full_name
    }

    Category {
        UUID id PK
        varchar name
    }

    Book {
        UUID id PK
        UUID publisher_id FK
        varchar isbn UK
        varchar title
        int publication_year
        decimal price
    }

    CustomerOrder {
        UUID id PK
        UUID customer_id FK
        varchar status
        timestamp created_at
    }

    OrderItem {
        UUID id PK
        UUID order_id FK
        UUID book_id FK
        int quantity
        decimal unit_price
    }

    Review {
        UUID id PK
        UUID customer_id FK
        UUID book_id FK
        int rating
        varchar comment
        timestamp created_at
    }

    Publisher ||--o{ Book : publishes
    Author }|--o{ Book : writes
    Category }o--o{ Book : classifies
    Customer }o--o{ Book : wishlist
    Customer ||--o{ CustomerOrder : places
    CustomerOrder ||--|{ OrderItem : contains
    Book ||--o{ OrderItem : ordered_as
    Customer ||--o{ Review : writes
    Book ||--o{ Review : receives

    %% 9. кожна книга має щонайменше одного автора
    %% 10. status: одне зі значень new, paid, shipped, cancelled
    %% 11. rating: ціле від 1 до 5
    %% 12. унікальна пара: customer_id + book_id у Review
    %% 13. унікальна пара: order_id + book_id в OrderItem; quantity не менше 1.
    
Напиши  EARS-вимоги для онлайн-книгарні. Шаблони: WHEN <тригер> система SHALL <реакція>; WHILE <стан> система SHALL...; IF <небажана подія> THEN система SHALL...; WHERE <опція> система SHALL...
Кожна вимога має бути перевірюваною: конкретні числа, статуси, умови. Не використовуй слів "швидко", "коректно", "за потреби".
Обов'язково охопи: пошук, реєстрацію, кошик, оформлення замовлення, оплату через зовнішній платіжний шлюз (успіх, відмова, таймаут), доставку через зовнішню службу, скасування залежно від статусу, відгуки, керування книгами адміністратором, одну нефункціональну вимогу (безпека паролів).
Використовуй назви сутностей і статусів з моєї моделі. Формат: таблиця | ID (FR-01...) | Вимога |.

## Відповідь GPT (оригінал)
Ось  12 EARS-вимог для онлайн-книгарні, які охоплюють усі обов’язкові функції.

| ID    | Вимога                                                                                                                                                                                                   |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FR-01 | WHEN Гість або Покупець вводить пошуковий запит, система SHALL відображати Book, у яких `title` або `isbn` містить цей запит.                                                                            |
| FR-02 | WHEN Гість реєструється з унікальним `email`, система SHALL створювати Customer з `id`, `email` і `full_name`.                                                                                           |
| FR-03 | WHEN Покупець додає Book до кошика, система SHALL зберігати `book_id`, `quantity ≥ 1` і ціну книги.                                                                                                      |
| FR-04 | WHEN Покупець оформлює замовлення, система SHALL створювати CustomerOrder зі статусом `new` і пов’язаними OrderItem, у яких `quantity ≥ 1`, а пара `order_id + book_id` є унікальною.                    |
| FR-05 | WHEN Покупець оплачує CustomerOrder, система SHALL надсилати платіжному шлюзу ідентифікатор замовлення та суму до оплати.                                                                                |
| FR-06 | WHEN платіжний шлюз підтверджує успішну оплату, система SHALL змінювати `CustomerOrder.status` на `paid`.                                                                                                |
| FR-07 | IF платіжний шлюз відхиляє оплату або не відповідає протягом 30 секунд, THEN система SHALL залишати `CustomerOrder.status = new` і повідомляти Покупця про відмову або необхідність перевірки платежу.   |
| FR-08 | WHEN CustomerOrder має статус `paid`, система SHALL передавати дані замовлення службі доставки, а після підтвердження відправлення змінювати статус на `shipped`.                                        |
| FR-09 | WHEN Покупець скасовує CustomerOrder зі статусом `new`, система SHALL змінювати його статус на `cancelled`; замовлення зі статусом `paid` або `shipped` не можна скасувати через цей механізм.           |
| FR-10 | WHEN Покупець створює Review для Book, система SHALL приймати `rating` лише від 1 до 5 та забороняти повторний Review для тієї самої пари `customer_id + book_id`.                                       |
| FR-11 | WHEN Адміністратор створює або редагує Book, система SHALL зберігати `title`, `isbn`, `publication_year`, `price`, `publisher_id` і зв’язок щонайменше з одним Author, забезпечуючи унікальність `isbn`. |
| FR-12 | WHILE система зберігає пароль Customer, система SHALL використовувати Argon2id для хешування пароля з унікальною сіллю та не зберігати пароль у відкритому вигляді.                                      |

Примітка: кошик не є окремою сутністю в твоїй ER-моделі, тому його зберігання потрібно реалізувати окремо або тимчасово зберігати дані кошика в сесії.