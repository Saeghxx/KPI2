# Spec — Завдання 3: Взаємодія (Sequence)

## Обраний UC
UC-06 «Оплата замовлення» (Завдання 2). Мета актора: Покупець хоче сплатити CustomerOrder зі статусом `new` через платіжний шлюз і дізнатися результат. Діаграми будуються для сценаріїв SC-09–SC-12 (Gherkin) і узгоджуються з FR-05–FR-07 та ER-моделлю `bookstore.mmd`. 

## Учасники
| Учасник | Тип | Сутність ER / роль |
|---|---|---|
| Buyer | актор | Customer |
| UI/Controller | інтерфейс | — |
| OrderService | сервіс | CustomerOrder, OrderItem |
| PaymentService | сервіс | Payment |
| Gateway | зовнішня система | — |
| DB | сховище | CustomerOrder, OrderItem, Payment |

Скорочення: Buyer — Покупець, Gateway — платіжний шлюз, DB — база даних.