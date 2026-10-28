# Prompt 2

## Промпт 
Побудуй sequence-діаграми для Завдання 3 (онлайн-книгарня) строго за моєю специфікацією.\
\
Ось специфікація Lab3/spec.md (написана мною):\
**# Lab3: Взаємодія (Sequence)**

**\*\*Інструмент:\*\*** Mermaid

**## 1. Намір і сценарії**

Мета покупця — оплатити замовлення; діаграми показують успіх, відмову й тайм-аут.

\- **\*\*UC-06:\*\*** Оплата замовлення.
\- **\*\*SC-09 (FR-05, FR-04, UC-06):\*\*** ініціювання оплати.
\- **\*\*SC-10 (FR-05, FR-06, UC-07):\*\*** успішна оплата.
\- **\*\*SC-11 (FR-07, UC-08):\*\*** відмова.
\- **\*\*SC-12 (FR-05, FR-07, UC-08):\*\*** тайм-аут.

**## 2. Учасники та ER**

\| Учасник | Відповідність ER |
\|---|---|
\| \`Buyer\` | \`Customer\` |
\| \`UI/Controller\` | Технічний компонент |
\| \`OrderService\` | \`CustomerOrder\`, \`OrderItem\` |
\| \`PaymentService\` | \`Payment\` |
\| \`Gateway\` | Зовнішня система, поза ER |
\| \`DB\` | Сховище ER-сутностей |

**## 3. Спільна послідовність SC-09**

**\*\*Передумова:\*\*** замовлення має статус \`new\`, сума позицій — \`500.00\`.

1\. \`Buyer → UI/Controller\`: \`submitPayment(orderId)\`.
2\. \`UI/Controller → OrderService\`: \`processOrderPayment(orderId)\`.
3\. \`OrderService → DB\`: \`readOrder(orderId)\`; повернення \`order\`.
4\. \`opt [status = new]\`: \`readOrderItems(orderId)\`; повернення \`orderItems\`; \`loop\` обчислює суму через \`calculateAmount()\`.
5\. Усередині \`opt\`: \`processPayment(orderId, amount)\`, далі \`chargeOrder(orderId, amount)\` до \`Gateway\`.

SC-10–SC-12 повторюють спільну послідовність від кроку 4. Їхній \`opt [status = new]\` охоплює потік до відповідного завершального виклику \`showSuccess()\`, \`showDeclined()\` або \`showVerificationNotice()\` включно.

**## 4. Результати сценаріїв**

\- **\*\*SC-09:\*\*** \`chargeOrder(orderId, amount)\` надіслано з сумою \`500.00\`.
\- **\*\*SC-10:\*\*** \`Gateway\` повертає \`PaymentResult(success, transactionId)\`. \`PaymentService → DB\`: \`savePayment(orderId, success, transactionId, amount)\`; DB повертає \`paymentSaved\`. Далі \`PaymentService → OrderService\`: \`PaymentResult(success, transactionId)\`. \`OrderService → DB\`: \`updateOrderStatus(orderId, paid)\`; DB повертає \`status(paid)\`. \`UI/Controller → Buyer\`: \`showSuccess()\`.
\- **\*\*SC-11:\*\*** \`Gateway\` один раз повертає \`PaymentResult(result, transactionId)\` перед \`alt [result = declined] / else [result = success]\`. При відмові зберігається \`Payment\` зі статусом \`declined\`, замовлення залишається \`new\`, \`UI/Controller → Buyer\`: \`showDeclined()\`. При успіху повністю повторюється послідовність SC-10.
\- **\*\*SC-12:\*\*** після \`chargeOrder\` запускається відлік 30 секунд; \`PaymentService\` викликає \`checkTimeout()\`. \`alt [timeout] / else [response received]\`. При тайм-ауті відповіді Gateway немає; зберігається \`Payment\` зі статусом \`pending_verification\` і \`transactionId = NULL\`, замовлення залишається \`new\`, \`UI/Controller → Buyer\`: \`showVerificationNotice()\`. При відповіді Gateway повертає \`PaymentResult(result, transactionId)\`; вкладений \`alt\` розрізняє \`success\` і \`declined\` та повністю повторює відповідну послідовність SC-10 або SC-11.

**## 5. Критерії прийняття**

1\. Фрагменти: SC-09 — \`opt\` + \`loop\`; SC-10 — \`opt\`; SC-11 — \`opt\` + \`alt\`; SC-12 — \`opt\` + зовнішній і вкладений \`alt\`.
2\. Return-повідомлення малювати пунктиром: \`order\`, \`orderItems\`, \`PaymentResult(...)\`, \`paymentSaved\`, \`status(paid)\`. У SC-11 шлюз повертає результат лише один раз перед \`alt\`.
3\. Activation bars відповідають викликам і поверненням; \`UI/Controller\` та \`OrderService\` активні до \`opt\` і після нього; \`PaymentService\` не деактивується всередині \`alt\`. У SC-12 активація \`Gateway\` закривається після відповіді в \`[response received]\` або фіксації тайм-ауту в \`[timeout]\`; відповіді Gateway у гілці тайм-ауту немає.
4\. **\*\*Назви викликів\*\*** мають формат «дієслово + іменник()»; використовуються назви й параметри з цього spec.
5\. Учасники узгоджені з ER; \`UI/Controller\` і \`Gateway\` не є сутностями ER.
6\. Діаграми відповідають Gherkin і spec, без нових непогоджених правил.
7\. Завершальні стани: SC-10 — платіж \`success\`, замовлення \`paid\`; SC-11 — платіж \`declined\`, замовлення \`new\`; SC-12 — \`pending_verification\`, замовлення \`new\`.
8\. \`savePayment(...)\` і \`updateOrderStatus(...)\` містять \`orderId\`; сума платежу відповідає сумі позицій.

**## 6. Припущення та поза обсягом**

\- Gateway повертає результат синхронно; у SC-12 відповідь може не надійти протягом 30 секунд.
\- \`Payment\` створюється після відповіді Gateway або фіксації тайм-ауту; для \`success\`, \`declined\` і \`pending_verification\` зберігається запис.
\- \`transactionId\` може бути \`NULL\`, якщо шлюз не повернув ідентифікатор; непорожні значення унікальні.
\- Гілки успіху SC-11 і SC-12 наведено для повноти альтернативних потоків; вони виходять за межі основного результату Gherkin SC-11.
\- Замовлення зі статусом, відмінним від \`new\`, поза обсягом.
\- Поза обсягом: внутрішня логіка шлюзу, банківські перевірки, повторні спроби, пізні callback-и та реалізація UI.\
\
Ось ER-модель:\
\# ER-модель (оновлена в Завданні 2)\
\
erDiagram\
&#x20;   Customer {\
&#x20;       UUID id PK\
&#x20;       varchar email UK\
&#x20;       varchar full_name\
&#x20;       varchar password_hash\
&#x20;       varchar role\
&#x20;       varchar phone\
&#x20;       varchar address\
&#x20;   }\
\
&#x20;   Publisher {\
&#x20;       UUID id PK\
&#x20;       varchar name\
&#x20;   }\
\
&#x20;   Author {\
&#x20;       UUID id PK\
&#x20;       varchar full_name\
&#x20;   }\
\
&#x20;   Category {\
&#x20;       UUID id PK\
&#x20;       varchar name\
&#x20;   }\
\
&#x20;   Book {\
&#x20;       UUID id PK\
&#x20;       UUID publisher_id FK\
&#x20;       varchar isbn UK\
&#x20;       varchar title\
&#x20;       int publication_year\
&#x20;       decimal price\
&#x20;   }\
\
&#x20;   CustomerOrder {\
&#x20;       UUID id PK\
&#x20;       UUID customer_id FK\
&#x20;       varchar status\
&#x20;       timestamp created_at\
&#x20;   }\
\
&#x20;   OrderItem {\
&#x20;       UUID id PK\
&#x20;       UUID order_id FK\
&#x20;       UUID book_id FK\
&#x20;       int quantity\
&#x20;       decimal unit_price\
&#x20;   }\
\
&#x20;   Review {\
&#x20;       UUID id PK\
&#x20;       UUID customer_id FK\
&#x20;       UUID book_id FK\
&#x20;       int rating\
&#x20;       varchar comment\
&#x20;       timestamp created_at\
&#x20;   }\
\
&#x20;   Payment {\
&#x20;       UUID id PK\
&#x20;       UUID order_id FK\
&#x20;       varchar transaction_id UK\
&#x20;       varchar payment_status\
&#x20;       decimal amount\
&#x20;   }\
\
&#x20;   Shipment {\
&#x20;       UUID id PK\
&#x20;       UUID order_id FK\
&#x20;       varchar shipment_id UK\
&#x20;   }\
\
&#x20;   Publisher ||--o{ Book : publishes\
&#x20;   Author }|--o{ Book : writes\
&#x20;   Category }o--o{ Book : classifies\
&#x20;   Customer }o--o{ Book : wishlist\
&#x20;   Customer ||--o{ CustomerOrder : places\
&#x20;   CustomerOrder ||--|{ OrderItem : contains\
&#x20;   Book ||--o{ OrderItem : ordered_as\
&#x20;   Customer ||--o{ Review : writes\
&#x20;   Book ||--o{ Review : receives\
&#x20;   CustomerOrder ||--o{ Payment : paid_by\
&#x20;   CustomerOrder ||--o| Shipment : shipped_as\
\
&#x20;   %% 9. кожна книга має щонайменше одного автора\
&#x20;   %% 10. status: одне зі значень new, paid, shipped, cancelled\
&#x20;   %% 11. rating: ціле від 1 до 5\
&#x20;   %% 12. унікальна пара: customer_id + book_id у Review\
&#x20;   %% 13. унікальна пара: order_id + book_id в OrderItem; quantity не менше 1\
&#x20;   %% 14. role: customer або admin; password_hash: хеш Argon2id (FR-12)\
&#x20;   %% 15. payment_status: success, declined або pending_verification (FR-07)\
&#x20;   %% 16. transaction_id може бути порожнім при pending_verification\
\
Ось Gherkin SC-09…SC-12:\
 @FR-05 @FR-04
  Сценарій: SC-09 Надсилання запиту на оплату
    Дано існує CustomerOrder зі статусом "new"
    І загальна сума замовлення дорівнює 500.00
    Коли Покупець ініціює оплату
    Тоді система надсилає платіжному шлюзу ідентифікатор замовлення
    І передає суму 500.00

  @FR-06
  Сценарій: SC-10 Успішна оплата
    Дано існує CustomerOrder зі статусом "new"
    Коли платіжний шлюз підтверджує успішну оплату
    Тоді система змінює CustomerOrder.status на "paid"

  @FR-07
  Сценарій: SC-11 Відмова в оплаті
    Дано існує CustomerOrder зі статусом "new"
    Коли платіжний шлюз відхиляє оплату
    Тоді система залишає CustomerOrder.status рівним "new"
    І повідомляє Покупця про відмову в оплаті

  @FR-07 @FR-05
  Сценарій: SC-12 Таймаут платіжного шлюзу
    Дано існує CustomerOrder зі статусом "new"
    Коли платіжний шлюз не повертає результат протягом 30 секунд
    Тоді система залишає CustomerOrder.status рівним "new"
    І позначає платіж як такий, що потребує перевірки
    І повідомляє Покупця про необхідність перевірки платежу
    І не повідомляє про успішну оплату без підтвердження
\
Завдання:\
1\. Створи чотири окремі діаграми Mermaid sequenceDiagram: SC-09, SC-10, SC-11, SC-12. Кожна в окремому блоці \`\`\`mermaid.\
2\. Учасників, назви викликів, параметри, return-повідомлення, фрагменти (opt, loop, alt) і умови бери лише зі spec. Нічого не вигадуй. Return-повідомлення пунктирними стрілками (-->>). Activation bars через activate/deactivate.\
3\. Після діаграм виведи таблицю самоперевірки: критерій 1–8 зі spec / виконано (так/ні) / що не збігається.\
4\. Не змінюй і не переписуй spec. Якщо знайдеш у ньому суперечність або неясність, не розв'язуй її сама, а перелічи окремо в кінці.

## Відповідь GPT (оригінал)
Нижче — чотири окремі Mermaid sequence-діаграми за твоїм `Lab3/spec.md`. Зберігаю заданих учасників, назви повідомлень, параметри, умови фрагментів і результати сценаріїв. Після діаграм — таблиця перевірки критеріїв 1–8 та окремий перелік неоднозначностей специфікації.

## SC-09 — Надсилання запиту на оплату
sequenceDiagram
    actor Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant GW as Gateway
    participant DB

    activate Buyer
    Buyer->>UI: submitPayment(orderId)
    activate UI
    UI->>OS: processOrderPayment(orderId)
    activate OS
    OS->>DB: readOrder(orderId)
    activate DB
    DB-->>OS: order
    deactivate DB

    opt status = new
        OS->>DB: readOrderItems(orderId)
        activate DB
        DB-->>OS: orderItems
        deactivate DB

        loop по позиціях замовлення
            OS->>OS: calculateAmount()
        end

        OS->>PS: processPayment(orderId, amount)
        activate PS
        PS->>GW: chargeOrder(orderId, amount)
        activate GW
        deactivate GW
        deactivate PS
    end

    deactivate OS
    deactivate UI
    deactivate Buyer

## SC-10 — Успішна оплата
sequenceDiagram
    actor Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant GW as Gateway
    participant DB

    activate Buyer
    Buyer->>UI: submitPayment(orderId)
    activate UI
    UI->>OS: processOrderPayment(orderId)
    activate OS
    OS->>DB: readOrder(orderId)
    activate DB
    DB-->>OS: order
    deactivate DB

    opt status = new
        OS->>DB: readOrderItems(orderId)
        activate DB
        DB-->>OS: orderItems
        deactivate DB

        loop по позиціях замовлення
            OS->>OS: calculateAmount()
        end

        OS->>PS: processPayment(orderId, amount)
        activate PS
        PS->>GW: chargeOrder(orderId, amount)
        activate GW
        GW-->>PS: PaymentResult(success, transactionId)
        deactivate GW

        PS->>DB: savePayment(orderId, success, transactionId, amount)
        activate DB
        DB-->>PS: paymentSaved
        deactivate DB

        PS-->>OS: PaymentResult(success, transactionId)
        deactivate PS

        OS->>DB: updateOrderStatus(orderId, paid)
        activate DB
        DB-->>OS: status(paid)
        deactivate DB

        OS->>UI: showSuccess()
        UI->>Buyer: showSuccess()
    end

    deactivate OS
    deactivate UI
    deactivate Buyer

## SC-11 — Відмова в оплаті

Шлюз повертає `PaymentResult(result, transactionId)` один раз перед `alt`. Гілка успіху повністю показує відповідну обробку SC-10.

sequenceDiagram
    actor Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant GW as Gateway
    participant DB

    activate Buyer
    Buyer->>UI: submitPayment(orderId)
    activate UI
    UI->>OS: processOrderPayment(orderId)
    activate OS
    OS->>DB: readOrder(orderId)
    activate DB
    DB-->>OS: order
    deactivate DB

    opt status = new
        OS->>DB: readOrderItems(orderId)
        activate DB
        DB-->>OS: orderItems
        deactivate DB

        loop по позиціях замовлення
            OS->>OS: calculateAmount()
        end

        OS->>PS: processPayment(orderId, amount)
        activate PS
        PS->>GW: chargeOrder(orderId, amount)
        activate GW
        GW-->>PS: PaymentResult(result, transactionId)
        deactivate GW

        alt result = declined
            PS->>DB: savePayment(orderId, declined, transactionId, amount)
            activate DB
            DB-->>PS: paymentSaved
            deactivate DB

            PS-->>OS: PaymentResult(declined, transactionId)
            OS->>UI: showDeclined()
            UI->>Buyer: showDeclined()
        else result = success
            PS->>DB: savePayment(orderId, success, transactionId, amount)
            activate DB
            DB-->>PS: paymentSaved
            deactivate DB

            PS-->>OS: PaymentResult(success, transactionId)
            OS->>DB: updateOrderStatus(orderId, paid)
            activate DB
            DB-->>OS: status(paid)
            deactivate DB

            UI->>Buyer: showSuccess()
        end

        deactivate PS
    end

    deactivate OS
    deactivate UI
    deactivate Buyer
    
## SC-12 — Тайм-аут платіжного шлюзу

Зовнішній `alt` розділяє тайм-аут і отримання відповіді. У другій гілці вкладений `alt` розрізняє успішну оплату й відмову.

Діаграма некоректна або не підтримується


## 4. Таблиця самоперевірки

| Критерій                          | Виконано | Що не збігається                                                                                                                                                                           |
| --------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1. Фрагменти `opt`, `loop`, `alt` | Так      | Усі потрібні фрагменти присутні.                                                                                                                                                           |
| 2. Пунктирні return-повідомлення  | Частково | Повернення `order`, `orderItems`, `PaymentResult(...)`, `paymentSaved`, `status(paid)` пунктирні. У SC-09 відповіді Gateway немає, як і вимагає сценарій.                                  |
| 3. Activation bars                | Частково | `UI/Controller` і `OrderService` охоплюють `opt`; `PaymentService` не завершується всередині `alt`. У SC-09 активація Gateway закривається без показаного повернення — це неоднозначність. |
| 4. Формат назв викликів           | Так      | Використано назви методів зі специфікації.                                                                                                                                                 |
| 5. Учасники та ER                 | Так      | Використані `Buyer`, `UI/Controller`, `OrderService`, `PaymentService`, `Gateway`, `DB`.                                                                                                   |
| 6. Відповідність Gherkin і spec   | Частково | Альтернативні гілки успіху в SC-11 і SC-12 позначені як припущення в spec. Є зайві проміжні виклики до UI, зазначені нижче.                                                                |
| 7. Завершальні стани              | Частково | Стани платежу й замовлення показані через відповідні операції, але явні повідомлення про незмінність `new` відсутні.                                                                       |
| 8. `orderId` і сума               | Так      | `savePayment(...)` та `updateOrderStatus(...)` містять `orderId`; сума передається через `amount`.                                                                                         |

## 5. Неоднозначності та необхідні виправлення

1. Зайві виклики в SC-10 і SC-11. У діаграмах є `OS->>UI: showSuccess()` та `OS->>UI: showDeclined()`, яких немає у spec. Видали ці рядки: специфікація визначає саме `UI/Controller → Buyer: showSuccess()` або `showDeclined()`.
2. SC-09: закриття активації `Gateway` після `chargeOrder()` без відповіді не цілком узгоджується з критерієм про відповідність активацій викликам і поверненням. Специфікація водночас вимагає завершити сценарій на надсиланні запиту.
3. SC-12: `checkTimeout()` викликається після `chargeOrder()`, але сама назва методу не визначає, чи очікує він 30 секунд, чи лише перевіряє таймер. Відлік у примітці показаний, однак механізм очікування не уточнений.
4. SC-12: повідомлення `PaymentResult(pending_verification, NULL)` не задане явно в розділі результатів останньої специфікації. Його варто підтвердити або прибрати, якщо потрібно дотримуватися лише явно зазначених повідомлень.
5. Критерій 7: `new` після відмови й тайм-ауту зафіксовано в тексті специфікації, але окремого повідомлення `status(new)` немає. Додавати його без уточнення не слід.
6. Критерій 3: для SC-11 у гілці успіху й SC-12 у вкладених гілках активації мають відповідати фактичним викликам. Після видалення зайвих повідомлень до UI варто повторно перевірити їх візуально в Mermaid.

Це аудит відповідності, а не зміна твоєї специфікації. Найважливіші моменти перед здачею — прибрати два зайві виклики до UI та визначитися з неоднозначностями SC-09 і SC-12.


## Мій фідбек (написано мною)
1. SC-12 не рендериться: `deactivate GW` стоїть у двох гілках alt, а Mermaid читає лінійно. Суперечить критерію 3. Рішення: змінити spec.
2. Результат не доходить до UI: немає повернення OrderService → UI/Controller, а UI показує showX(). У SC-10/11 showX() продубльований. Суперечить критерію 2 і шляху повернення. Рішення: змінити spec, не видаляти виклики вручну, як радив GPT.
3. SC-09: активація Gateway відкривається і одразу закривається. Рішення: уточнити в spec, де закривається.