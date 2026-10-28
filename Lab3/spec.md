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

### SC-10 — Успішна оплата (FR-05, FR-06, UC-07)
Кроки 1–7 як у SC-09, далі:
8. Gateway ⇢ PaymentService: `PaymentResult(success, transactionId)`.
9. PaymentService → DB: `savePayment(orderId, success, transactionId, amount)`; DB ⇢ PaymentService: `paymentSaved`.
10. PaymentService ⇢ OrderService: `PaymentResult(success)`.
11. OrderService → DB: `updateOrderStatus(orderId, paid)`; DB ⇢ OrderService: `status(paid)`.
12. OrderService ⇢ UI/Controller ⇢ Buyer: `showSuccess()`.

### SC-11 — Відмова в оплаті (FR-07, UC-08)
Кроки 1–7 як у SC-09, далі:
8. Gateway ⇢ PaymentService: `PaymentResult(declined, transactionId)`.
9. PaymentService → DB: `savePayment(orderId, declined, transactionId, amount)`; DB ⇢ PaymentService: `paymentSaved`.
10. PaymentService ⇢ OrderService: `PaymentResult(declined)`. `updateOrderStatus` не викликається, статус лишається `new`.
11. OrderService ⇢ UI/Controller ⇢ Buyer: `showDeclined()`.