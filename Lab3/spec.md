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


## Сценарії

### SC-09 — Надсилання запиту на оплату (FR-05, UC-06)
Передумова: `status = new`, сума тестового замовлення 500.00 UAH.
1. Buyer → UI/Controller: `submitPayment(orderId)`.
2. UI/Controller → OrderService: `processOrderPayment(orderId)`.
3. OrderService → DB: `readOrder(orderId)`; DB ⇢ OrderService: `order`.
4. OrderService → DB: `readOrderItems(orderId)`; DB ⇢ OrderService: `orderItems`.
5. OrderService: `calculateAmount()` — у `loop` по позиціях, `amount = Σ(quantity × unit_price)`.
6. OrderService → PaymentService: `processPayment(orderId, amount)`.
7. PaymentService → Gateway: `chargeOrder(orderId, amount)`.

SC-09 закінчується на кроці 7: Gherkin далі не йде.