# Промпт 4 — верифікація виходу

## Промпт
Перевір model/bookstore.mmd проти кожного критерію spec.md (1–8). Для кожного — виконано/ні та де саме.

## Результат перевірки
| № | Критерій | Статус | Де перевірено |
|---|----------|--------|---------------|
| 1 | `UUID` + `PK` на всіх `id` | ✔ | 8 сутностей, 8 `PK` |
| 2 | 3NF, без похідних | ✔ | `total` відсутній; `unit_price` — знімок ціни, не похідне |
| 3 | N:M напряму | ✔ | Author–Book, Category–Book, Customer–Book; OrderItem має `quantity`, `unit_price` |
| 4 | Mermaid erDiagram | ✔ | рендериться |
| 5 | `FK` на боці «багато» | ✔ | `publisher_id`, `customer_id`, `order_id`, `book_id` |
| 6 | `timestamp` | ✔ | `created_at` ×2 |
| 7 | `decimal`, імена збігаються | ✔ | `price`, `unit_price` |
| 8 | `UK` | ✔ | `email`, `isbn` |
