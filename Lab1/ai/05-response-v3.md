# Відповідь ChatGPT на промпт 5 (v3, після spec v5)

# 1. Мій промпт
Перегенеруй лише коментарі до діаграми для обмежень 9-13.\
Діаграму (сутності, типи, зв'язки, PK/FK/UK) не змінюй.\
\
Вимоги до коментарів:\
\- словами, без SQL-ключових слів і без синтаксису з дужками;\
\- для критерію 12 формулюй так: «унікальна пара: customer_id + book_id у Review»;\
\- для критерію 13: «унікальна пара: order_id + book_id в OrderItem; quantity не менше 1»;\
\- для критерію 11: «rating: ціле від 1 до 5»;\
\- для критерію 10: «status: одне зі значень new, paid, shipped, cancelled»;\
\- для критерію 9: «кожна книга має щонайменше одного автора».\
\
Поверни повний код діаграми (від erDiagram до останнього коментаря),\
а потім таблицю перевірки критеріїв 1-14.

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
14\. Обмеження 9-13, які Mermaid не виражає, описуються короткими
    коментарями в діаграмі або таблицею в spec. Вони формулюються
    словами («унікальна пара X, Y»), без SQL-ключових слів UNIQUE і CHECK
    та без SQL-синтаксису (дужок, назв таблиць із крапкою).

    **## Поза обсягом**
\- SQL DDL (CREATE TABLE, CHECK, UNIQUE тощо)
\- ORM-моделі
\- Індекси
\- Автентифікація та авторизація
\- ОплатаSpec оновлено: критерій 14 тепер забороняє SQL-терміни UNIQUE і CHECK\
у коментарях, а також додано розділ «Поза обсягом».\
\

# 2. Відповідь AI

## Mermaid ER-діаграма

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

## Таблиця перевірки критеріїв 1–14

| №  | Критерій                                                                 | Статус                                                                       |
| -- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| 1  | Усі `id` мають тип `UUID` і позначку `PK`                                | Виконано                                                                     |
| 2  | Модель у 3NF, немає похідних атрибутів і повторюваних груп               | Виконано на рівні структури                                                  |
| 3  | Зв’язки N:M показані безпосередньо, без сполучних таблиць                | Виконано                                                                     |
| 4  | Формат Mermaid `erDiagram`                                               | Виконано                                                                     |
| 5  | Зовнішні ключі на стороні «багато» позначені `FK`                        | Виконано                                                                     |
| 6  | Часові поля мають тип `timestamp`                                        | Виконано                                                                     |
| 7  | Грошові поля мають тип `decimal`, назви `price` і `unit_price` збережено | Виконано                                                                     |
| 8  | `email` та `isbn` позначені `UK`                                         | Виконано                                                                     |
| 9  | Кожна книга має щонайменше одного автора                                 | Задокументовано коментарем; фактичне забезпечення потребує додаткової логіки |
| 10 | Допустимі значення `status`                                              | Задокументовано коментарем                                                   |
| 11 | `rating` — ціле від 1 до 5                                               | Задокументовано коментарем                                                   |
| 12 | Один відгук покупця на одну книгу                                        | Задокументовано коментарем                                                   |
| 13 | Одна позиція книги в замовленні та `quantity` не менше 1                 | Задокументовано коментарем                                                   |
| 14 | Обмеження 9–13 сформульовані словами, без SQL-синтаксису в коментарях    | Виконано                                                                     |

Примітка: структура діаграми, сутності, типи, зв’язки та позначки ключів не змінені. Перероблено лише коментарі до обмежень 9–13 відповідно до нових вимог.