# Prompt 1 — Актори і цілі

## Промпт
Домен: онлайн-книгарня. Ось моя ER-модель: erDiagram
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
    
Запропонуй акторів: щонайменше одну людину і дві зовнішні системи (платіжний шлюз, служба доставки). Для кожного вкажи тип і головну ціль. Окремо вкажи, хто є узагальненням кого (наприклад, Покупець є Гість). Відповідь у вигляді таблиці Markdown.

## Відповідь GPT (оригінал)

| Актор | Тип актора | Головна ціль |
|---|---|---|
| Гість | Людина | Переглядати каталог книг, шукати книги та переглядати інформацію про них без реєстрації. |
| Покупець | Людина | Реєструватися, оформлювати замовлення, оплачувати книги, залишати відгуки та додавати книги до списку бажаного. |
| Адміністратор | Людина | Керувати каталогом книг, авторами, видавництвами, категоріями та контролювати замовлення. |
| Платіжний шлюз | Зовнішня система | Обробляти платежі покупців і повертати результат оплати онлайн-книгарні. |
| Служба доставки | Зовнішня система | Отримувати дані про замовлення та доставку, забезпечувати доставку книг покупцям і передавати статус доставки. |

Узагальнення акторів:

| Базовий актор | Спеціалізований актор | Пояснення |
|---|---|---|
| Гість | Покупець | Покупець є зареєстрованим користувачем, який має додаткові можливості порівняно з гостем. |
| Користувач системи | Гість | Гість може переглядати каталог без реєстрації. |
| Користувач системи | Покупець | Покупець має доступ до функцій оформлення замовлень, оплати та написання відгуків. |

## Що виправлено вручну
- Прибрано абстрактного актора «Користувач системи»: Покупець мав двох батьків (Гість і Користувач системи), що суперечить узагальненню. Залишено одну лінію: Покупець є Гість.
- Прибрано примітку GPT про «варіант використання узагальнення»: вона плутала узагальнення акторів із узагальненням use case.
- Звірено цілі акторів із сутностями ER (Wishlist, Publisher, Author, Category).