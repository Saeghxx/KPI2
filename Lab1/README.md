# Онлайн-магазин книг

Проєкт моделює дані онлайн-магазину книг. Покупець переглядає каталог (книги, автори, видавництва, категорії), додає книги до списку бажаного, оформлює замовлення та залишає відгуки з оцінкою.

Обсяг завдання 1: лише ER-модель (Mermaid `erDiagram`), без SQL DDL та ORM.

## Структура
- `spec.md` — сутності, атрибути, зв'язки, критерії прийняття
- `model/bookstore.mmd` — ER-модель (джерело) + `model/bookstore.png` — рендер
- `ai/` — повний слід промптів і аудит виходу AI
- `adr/` — рішення (MADR)
- `DEFENSE.md` — точка входу для рев'ю

## ER-діаграма

```mermaid
erDiagram
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
    %% 13. унікальна пара: order_id + book_id в OrderItem; quantity не менше 1
    ```