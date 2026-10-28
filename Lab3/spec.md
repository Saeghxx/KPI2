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

### SC-12 — Таймаут оплати (FR-05, FR-07, UC-08)
Кроки 1–7 як у SC-09, далі:
8. Відповіді на `chargeOrder` немає 30 секунд: PaymentService виконує `checkTimeout()` і визначає, що час вийшов.
9. PaymentService → DB: `savePayment(orderId, pending_verification, null, amount)`; DB ⇢ PaymentService: `paymentSaved`.
10. PaymentService ⇢ OrderService: `PaymentResult(pending_verification)`. Статус лишається `new`.
11. OrderService ⇢ UI/Controller ⇢ Buyer: `showVerificationNotice()`. Успіх не повідомляється.

## Фрагменти
- Усі діаграми: `opt [status = new]` починається одразу після `readOrder` (крок 3) і охоплює решту потоку. Статус відомий лише після читання замовлення.
- SC-09: додатково `loop` для суми по позиціях замовлення.
- SC-11: `alt [відмова]` / `else [успіх]`; в `else` лише примітка «успіх — див. SC-10».
- SC-12: `alt [таймаут 30 с]` / `else [відповідь отримано]`; в `else` лише примітка «результат обробляється як у SC-10/SC-11».
- Activation bars UI/Controller і OrderService починаються до `opt` і закінчуються після нього. Activation bar PaymentService не обривається всередині `alt`.
- Пунктирні стрілки позначають return-повідомлення.

## Припущення моделювання (не кроки Gherkin)
- `opt [status = new]` і `else`-гілки (у Gherkin статус — лише передумова «Дано»).
- Шлюз повертає результат синхронною відповіддю на `chargeOrder()`.
- Таймаут визначає сам PaymentService (`checkTimeout()`).
- `Payment` створюється після відповіді шлюзу або таймауту (в ER немає початкового статусу).
- Назва return-повідомлення `PaymentResult` і методів `calculateAmount()`, `checkTimeout()` — моє позначення.

## Критерії прийняття
1. SC-09 має `loop` і `opt`; SC-10 `opt`; SC-11 і SC-12 `opt` та `alt`.
2. Кожен виклик, де сценарій передбачає відповідь, має штрихове return-повідомлення.
3. Activation bar UI/Controller і OrderService починається до `opt` і закінчується після нього; PaymentService не обривається в `alt`.
4. Назви викликів мають вигляд дієслово + іменник().
5. Учасники відповідають таблиці й ER (Payment, CustomerOrder, OrderItem).
6. Кроки збігаються з Gherkin; усе, чого там немає, перелічено в «Припущеннях».
7. SC-09 закінчується на `chargeOrder`; SC-10 — `paid`; SC-11 — `declined`, статус `new`; SC-12 — `pending_verification`, статус `new`, успіх не повідомлено.
8. `savePayment` і `updateOrderStatus` містять `orderId`.

## Поза обсягом
Внутрішня робота шлюзу, банківські перевірки, повторні спроби оплати, пізній callback після таймауту, реалізація UI.