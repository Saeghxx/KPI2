# Lab3: Взаємодія (Sequence)

**Інструмент:** Mermaid

## 1. Намір і сценарії

Мета покупця — оплатити замовлення; діаграми показують успіх, відмову й тайм-аут.

- **UC-06:** Оплата замовлення.
- **SC-09 (FR-05, FR-04, UC-06):** ініціювання оплати.
- **SC-10 (FR-05, FR-06, UC-07):** успішна оплата.
- **SC-11 (FR-07, UC-08):** відмова.
- **SC-12 (FR-05, FR-07, UC-08):** тайм-аут.

## 2. Учасники та ER

| Учасник | Відповідність ER |
|---|---|
| `Buyer` | `Customer` |
| `UI/Controller` | Технічний компонент |
| `OrderService` | `CustomerOrder`, `OrderItem` |
| `PaymentService` | `Payment` |
| `Gateway` | Зовнішня система, поза ER |
| `DB` | Сховище ER-сутностей |

## 3. Спільна послідовність SC-09

**Передумова:** замовлення має статус `new`, сума позицій — `500.00`.

1. `Buyer → UI/Controller`: `submitPayment(orderId)`.
2. `UI/Controller → OrderService`: `processOrderPayment(orderId)`.
3. `OrderService → DB`: `readOrder(orderId)`; повернення `order`.
4. `opt [status = new]`: `readOrderItems(orderId)`; повернення `orderItems`; `loop` обчислює суму через `calculateAmount()`.
5. Усередині `opt`: `processPayment(orderId, amount)`, далі `chargeOrder(orderId, amount)` до `Gateway`.

SC-10–SC-12 повторюють спільну послідовність від кроку 4. Їхній `opt [status = new]` охоплює потік до відповідного завершального виклику `showSuccess()`, `showDeclined()` або `showVerificationNotice()` включно.

## 4. Результати сценаріїв

- **SC-09:** `chargeOrder(orderId, amount)` надіслано з сумою `500.00`.
- **SC-10:** `Gateway` повертає `PaymentResult(success, transactionId)`. `PaymentService → DB`: `savePayment(orderId, success, transactionId, amount)`; DB повертає `paymentSaved`. Далі `PaymentService → OrderService`: `PaymentResult(success, transactionId)`. `OrderService → DB`: `updateOrderStatus(orderId, paid)`; DB повертає `status(paid)`. `UI/Controller → Buyer`: `showSuccess()`.
- **SC-11:** `Gateway` один раз повертає `PaymentResult(result, transactionId)` перед `alt [result = declined] / else [result = success]`. При відмові зберігається `Payment` зі статусом `declined`, замовлення залишається `new`, `UI/Controller → Buyer`: `showDeclined()`. При успіху повністю повторюється послідовність SC-10.
- **SC-12:** після `chargeOrder` запускається відлік 30 секунд; `PaymentService` викликає `checkTimeout()`. `alt [timeout] / else [response received]`. При тайм-ауті відповіді Gateway немає; зберігається `Payment` зі статусом `pending_verification` і `transactionId = NULL`, замовлення залишається `new`, `UI/Controller → Buyer`: `showVerificationNotice()`. При відповіді Gateway повертає `PaymentResult(result, transactionId)`; вкладений `alt` розрізняє `success` і `declined` та повністю повторює відповідну послідовність SC-10 або SC-11.

## 5. Критерії прийняття

1. Фрагменти: SC-09 — `opt` + `loop`; SC-10 — `opt`; SC-11 — `opt` + `alt`; SC-12 — `opt` + зовнішній і вкладений `alt`.
2. Return-повідомлення малювати пунктиром: `order`, `orderItems`, `PaymentResult(...)`, `paymentSaved`, `status(paid)`. У SC-11 шлюз повертає результат лише один раз перед `alt`.
3. Activation bars відповідають викликам і поверненням; `UI/Controller` та `OrderService` активні до `opt` і після нього; `PaymentService` не деактивується всередині `alt`. У SC-12 активація `Gateway` закривається після відповіді в `[response received]` або фіксації тайм-ауту в `[timeout]`; відповіді Gateway у гілці тайм-ауту немає.
4. **Назви викликів** мають формат «дієслово + іменник()»; використовуються назви й параметри з цього spec.
5. Учасники узгоджені з ER; `UI/Controller` і `Gateway` не є сутностями ER.
6. Діаграми відповідають Gherkin і spec, без нових непогоджених правил.
7. Завершальні стани: SC-10 — платіж `success`, замовлення `paid`; SC-11 — платіж `declined`, замовлення `new`; SC-12 — `pending_verification`, замовлення `new`.
8. `savePayment(...)` і `updateOrderStatus(...)` містять `orderId`; сума платежу відповідає сумі позицій.

## 6. Припущення та поза обсягом

- Gateway повертає результат синхронно; у SC-12 відповідь може не надійти протягом 30 секунд.
- `Payment` створюється після відповіді Gateway або фіксації тайм-ауту; для `success`, `declined` і `pending_verification` зберігається запис.
- `transactionId` може бути `NULL`, якщо шлюз не повернув ідентифікатор; непорожні значення унікальні.
- Гілки успіху SC-11 і SC-12 наведено для повноти альтернативних потоків; вони виходять за межі основного результату Gherkin SC-11.
- Замовлення зі статусом, відмінним від `new`, поза обсягом.
- Поза обсягом: внутрішня логіка шлюзу, банківські перевірки, повторні спроби, пізні callback-и та реалізація UI.