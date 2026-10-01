# Онлайн-магазин книг — ER-модель

Завдання 1: домен і модель даних (ER).

## Домен

Онлайн-магазин книг: каталог (книги, автори, видавництва, категорії), клієнти, замовлення з позиціями та відгуки. Домен обрано тому, що в ньому є всі типові випадки ER-моделювання: 1:N (видавництво → книги), чисте M:N (книги ↔ автори, книги ↔ категорії) і M:N із власними атрибутами (замовлення ↔ книги, клієнти ↔ книги через відгуки). Предметна область зрозуміла без додаткових пояснень.

## Структура репозиторію

| Файл | Призначення |
|---|---|
| `spec.md` | Сутності, атрибути, зв'язки словами; критерії прийняття AC-1…AC-10 |
| `er/er.mmd` | ER-модель у декларативному синтаксисі Mermaid |
| `docs/ai-prompt.md` | Задача для ШІ, за якою згенеровано модель |
| `DEFENSE.md` | Захист: розбіжності, ADR, перевірка |

Фізичної схеми БД (SQL DDL, ORM) у репозиторії немає навмисно: це предмет курсу баз даних.

## Як це зроблено

1. Написано `spec.md` разом із критеріями прийняття.
2. ШІ згенерував `er/er.mmd` за промтом із `docs/ai-prompt.md` (ШІ використано, модель не ручна).
3. Аудит моделі проти `spec.md`; розбіжності виправлялися зміною spec/критеріїв з наступною перегенерацією, а не ручним патчем діаграми.

## ER-діаграма

Рендер ідентичний `er/er.mmd`.

```mermaid
erDiagram
    PUBLISHER ||--o{ BOOK : "видає"
    BOOK }o--o{ AUTHOR : "написана"
    BOOK }o--o{ CATEGORY : "належить до"
    CUSTOMER ||--o{ ORDER : "оформлює"
    ORDER ||--|{ ORDER_ITEM : "містить"
    BOOK ||--o{ ORDER_ITEM : "замовляється як"
    CUSTOMER ||--o{ REVIEW : "пише"
    BOOK ||--o{ REVIEW : "отримує"

    AUTHOR {
        int author_id PK
        string full_name
        int birth_year
        text biography
    }

    PUBLISHER {
        int publisher_id PK
        string name
        string country
    }

    CATEGORY {
        int category_id PK
        string name
    }

    BOOK {
        int book_id PK
        int publisher_id FK
        string isbn
        string title
        int publication_year
        string language
        decimal list_price
        int stock_qty
    }

    CUSTOMER {
        int customer_id PK
        string full_name
        string email
        string phone
        date registered_at
    }

    ORDER {
        int order_id PK
        int customer_id FK
        date placed_at
        string status
        string shipping_address
    }

    ORDER_ITEM {
        int order_id PK, FK
        int book_id PK, FK
        int quantity
        decimal unit_price
    }

    REVIEW {
        int customer_id PK, FK
        int book_id PK, FK
        int rating
        text comment
        date created_at
    }
```
