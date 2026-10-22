# Spec: онлайн-магазин книг (ER)

## Намір
Описати дані книжкового магазину: каталог, покупці, замовлення, відгуки, список бажаного.

## Сутності та атрибути
- **Customer**: `id`, `email` (унікальний), `full_name`.
- **Publisher**: `id`, `name`.
- **Author**: `id`, `full_name`.
- **Category**: `id`, `name`.
- **Book**: `id`, `publisher_id`, `isbn` (унікальний), `title`, `publication_year`, `price` (поточна ціна).
- **CustomerOrder**: `id`, `customer_id`, `status`, `created_at`.
- **OrderItem**: `id`, `order_id`, `book_id`, `quantity`, `unit_price` (ціна на момент замовлення).
- **Review**: `id`, `customer_id`, `book_id`, `rating`, `comment`, `created_at`.

## Зв'язки
- Publisher 1:N Book (видавництво випускає багато книг; книга має одне видавництво).
- Author N:M Book (співавторство).
- Category N:M Book.
- Customer N:M Book — «Список бажаного» (власних атрибутів немає).
- Customer 1:N CustomerOrder.
- CustomerOrder 1:N OrderItem; Book 1:N OrderItem. OrderItem — асоціативна сутність, бо зв'язок «замовлення–книга» несе власні атрибути (`quantity`, `unit_price`).
- Customer 1:N Review; Book 1:N Review.

## Критерії прийняття
1. Усі `id` мають тип `UUID` і позначені `PK`.
2. Модель у 3NF: немає похідних атрибутів (напр. сума замовлення) і груп, що повторюються.
3. Зв'язки N:M показані прямо між сутностями, без сполучних таблиць; асоціативна сутність — лише коли зв'язок має власні атрибути (OrderItem).
4. Формат — Mermaid `erDiagram`.
5. У зв'язках 1:N сутність на боці «багато» містить поле зовнішнього ключа з позначкою `FK`.
6. Поля з часом мають тип `timestamp`, не `datetime`.
7. Грошові поля мають тип `decimal`; імена полів у діаграмі збігаються зі spec дослівно (`price`, `unit_price`).
8. Природні унікальні ключі (`email`, `isbn`) позначені `UK`.
