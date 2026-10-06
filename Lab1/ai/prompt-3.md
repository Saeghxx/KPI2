# Промпт 3 — похідні поля, OrderItem, UK (spec v3 = spec.md)

## Промпт
spec оновлено: з CustomerOrder прибрано `total_price`; додано сутність OrderItem; критерії 2 і 3 уточнено, додано 7 і 8. Згенеруй діаграму ще раз, врахуй усі критерії 1–8.

(повний текст — `spec.md` цього репозиторію)

## Відповідь AI
```mermaid
erDiagram
    CUSTOMER {
        UUID id PK
        string email UK
        string full_name
    }
    PUBLISHER {
        UUID id PK
        string name
    }
    AUTHOR {
        UUID id PK
        string full_name
    }
    CATEGORY {
        UUID id PK
        string name
    }
    BOOK {
        UUID id PK
        UUID publisher_id FK
        string isbn UK
        string title
        int publication_year
        decimal price
    }
    CUSTOMER_ORDER {
        UUID id PK
        UUID customer_id FK
        string status
        timestamp created_at
    }
    ORDER_ITEM {
        UUID id PK
        UUID order_id FK
        UUID book_id FK
        int quantity
        decimal unit_price
    }
    REVIEW {
        UUID id PK
        UUID customer_id FK
        UUID book_id FK
        int rating
        string comment
        timestamp created_at
    }

    PUBLISHER ||--o{ BOOK : publishes
    AUTHOR }o--o{ BOOK : writes
    CATEGORY }o--o{ BOOK : classifies
    CUSTOMER }o--o{ BOOK : wishlists
    CUSTOMER ||--o{ CUSTOMER_ORDER : places
    CUSTOMER_ORDER ||--|{ ORDER_ITEM : contains
    BOOK ||--o{ ORDER_ITEM : "ordered as"
    CUSTOMER ||--o{ REVIEW : writes
    BOOK ||--o{ REVIEW : "reviewed in"
```

## Аудит
Усі розбіжності з prompt-1/2 закрито; перевірка по критеріях 1–8 — у `prompt-4.md`.
