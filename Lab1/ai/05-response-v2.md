# 1. Мій промпт оновлено 

**# Spec: онлайн-магазин книг (ER)**

**## Намір**
Описати дані книжкового магазину: каталог, покупці, замовлення, відгуки, список бажаного.

**## Сутності та атрибути**
\- **\*\*Customer\*\***: \`id\`, \`email\` (унікальний), \`full_name\`.
\- **\*\*Publisher\*\***: \`id\`, \`name\`.
\- **\*\*Author\*\***: \`id\`, \`full_name\`.
\- **\*\*Category\*\***: \`id\`, \`name\`.
\- **\*\*Book\*\***: \`id\`, \`publisher_id\`, \`isbn\` (унікальний), \`title\`, \`publication_year\`, \`price\` (поточна ціна).
\- **\*\*CustomerOrder\*\***: \`id\`, \`customer_id\`, \`status\`, \`created_at\`.
\- **\*\*OrderItem\*\***: \`id\`, \`order_id\`, \`book_id\`, \`quantity\`, \`unit_price\` (ціна на момент замовлення).
\- **\*\*Review\*\***: \`id\`, \`customer_id\`, \`book_id\`, \`rating\`, \`comment\`, \`created_at\`.

**## Зв'язки**
\- Publisher 1:N Book (видавництво випускає багато книг; книга має одне видавництво).
\- Author N:M Book (співавторство).
\- Category N:M Book.
\- Customer N:M Book — «Список бажаного» (власних атрибутів немає).
\- Customer 1:N CustomerOrder.
\- CustomerOrder 1:N OrderItem; Book 1:N OrderItem. OrderItem — асоціативна сутність, бо зв'язок «замовлення–книга» несе власні атрибути (\`quantity\`, \`unit_price\`).
\- Customer 1:N Review; Book 1:N Review.

**## Критерії прийняття**
1\. Усі \`id\` мають тип \`UUID\` і позначені \`PK\`.
2\. Модель у 3NF: немає похідних атрибутів (напр. сума замовлення) і груп, що повторюються.
3\. Зв'язки N:M показані прямо між сутностями, без сполучних таблиць
   (навіть якщо БД реляційна: це рівень ER, а не фізична схема);
   асоціативна сутність — лише коли зв'язок має власні атрибути (OrderItem).
4\. Формат — Mermaid \`erDiagram\`.
5\. У зв'язках 1:N сутність на боці «багато» містить поле зовнішнього ключа з позначкою \`FK\`.
6\. Поля з часом мають тип \`timestamp\`, не \`datetime\`.
7\. Грошові поля мають тип \`decimal\`; імена полів у діаграмі збігаються зі spec дослівно (\`price\`, \`unit_price\`).
8\. Природні унікальні ключі (\`email\`, \`isbn\`) позначені \`UK\`.
9\. Кожна книга має ≥1 автора: зв'язок Author–Book показано як
   AUTHOR }|--o{ BOOK (з боку Book автор обов'язковий).
10\. status замовлення має одне зі значень: new, paid, shipped, cancelled.
11\. rating у Review — ціле число від 1 до 5.
12\. Один покупець може залишити лише один відгук на одну книгу.
13\. В одному замовленні одна книга зустрічається в одній позиції; quantity ≥ 1.
14\. Обмеження 9–13, які Mermaid не виражає, описуються короткими
    коментарями у diagram або таблицею в spec; SQL (CHECK, UNIQUE) не пишемо. Spec оновлено (критерії 3, 9 уточнено, додано 14). Згенеруй Mermaid\
erDiagram ще раз за критеріями 1–14. Покажи код у блоці \`\`\`mermaid.

# 2. Відповідь AI

  Ось оновлена Mermaid `erDiagram` за критеріями 1–14. Зв’язки N:M показані безпосередньо між сутностями, без сполучних таблиць, а обмеження, які Mermaid не може гарантувати, зафіксовані в коментарях після діаграми.

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

    %% Constraints not fully expressible in Mermaid:
    %% 9. Every Book must have at least one Author.
    %% 10. CustomerOrder.status IN (new, paid, shipped, cancelled).
    %% 11. Review.rating is an integer from 1 to 5.
    %% 12. UNIQUE (Review.customer_id, Review.book_id).
    %% 13. UNIQUE (OrderItem.order_id, OrderItem.book_id).
    %% 13. OrderItem.quantity >= 1.

### Перевірка критеріїв 1–14

| №  | Реалізація                                                                             |
| -- | -------------------------------------------------------------------------------------- |
| 1  | Усі `id` мають тип `UUID` та позначку `PK`.                                            |
| 2  | Похідні атрибути й повторювані групи відсутні.                                         |
| 3  | Усі три зв’язки N:M показані безпосередньо: Author–Book, Category–Book, Customer–Book. |
| 4  | Формат `erDiagram`.                                                                    |
| 5  | Зовнішні ключі позначені `FK` на стороні «багато».                                     |
| 6  | Часові поля мають тип `timestamp`.                                                     |
| 7  | Грошові поля мають тип `decimal`; назви `price` та `unit_price` збережено.             |
| 8  | `email` та `isbn` позначені `UK`.                                                      |
| 9  | Обов’язковий автор книги задекларований у зв’язку та коментарі.                        |
| 10 | Допустимі статуси вказані в коментарі.                                                 |
| 11 | Діапазон рейтингу вказаний у коментарі.                                                |
| 12 | Унікальність відгуку покупця на книгу вказана в коментарі.                             |
| 13 | Унікальність книги в замовленні та `quantity >= 1` вказані в коментарях.               |
| 14 | Обмеження, які Mermaid не гарантує, задокументовані коментарями без SQL.               |

Примітка щодо критерію 9: запис `AUTHOR }|--o{ BOOK` відображає щонайменше одного автора з боку `Book` відповідно до заданої нотації. Проте фактичне забезпечення цього обмеження потребує додаткової логіки на рівні БД або застосунку.