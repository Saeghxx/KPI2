# Prompt 2

## Промпт
Побудуй sequence-діаграми для Завдання 3 (онлайн-книгарня) строго за моєю специфікацією.\
Це повторна генерація: попередня версія spec уточнена після аудиту діаграм.\
Змінено в spec: (1) додано повернення результату OrderService → UI/Controller у SC-10, SC-11, SC-12;\
(2) активація Gateway в SC-09 і SC-12 визначена явно (в SC-12 один deactivate після зовнішнього alt);\
(3) уточнено checkTimeout() і критерії 2–3.\
\
Ось специфікація Lab3/spec.md (написана мною):\
**## 3. Спільна послідовність SC-09**

**\*\*Передумова:\*\*** замовлення має статус \`new\`, сума позицій — \`500.00\`.

1\. \`Buyer → UI/Controller\`: \`submitPayment(orderId)\`.
2\. \`UI/Controller → OrderService\`: \`processOrderPayment(orderId)\`.
3\. \`OrderService → DB\`: \`readOrder(orderId)\`; повернення \`order\`.
4\. \`opt [status = new]\`: \`readOrderItems(orderId)\`; повернення \`orderItems\`; \`loop\` обчислює суму через \`calculateAmount()\`.
5\. Усередині \`opt\`: \`processPayment(orderId, amount)\`, далі \`chargeOrder(orderId, amount)\` до \`Gateway\`.

SC-10–SC-12 повторюють спільну послідовність від кроку 4. Їхній \`opt [status = new]\` охоплює потік до відповідного завершального виклику \`showSuccess()\`, \`showDeclined()\` або \`showVerificationNotice()\` включно.

У SC-09 \`Gateway\` активується під час виклику \`chargeOrder(orderId, amount)\` і залишається активним до завершення діаграми, оскільки в цьому сценарії відповідь від шлюзу не передбачена.

**## 4. Результати сценаріїв**

\- **\*\*SC-09:\*\*** \`chargeOrder(orderId, amount)\` надіслано з сумою \`500.00\`. Відповідь від \`Gateway\` у цьому сценарії не передбачена.

\- **\*\*SC-10:\*\*** \`Gateway\` повертає \`PaymentResult(success, transactionId)\`. \`PaymentService → DB\`: \`savePayment(orderId, success, transactionId, amount)\`; DB повертає \`paymentSaved\`. Далі \`PaymentService → OrderService\`: \`PaymentResult(success, transactionId)\`. \`OrderService → DB\`: \`updateOrderStatus(orderId, paid)\`; DB повертає \`status(paid)\`. Потім \`OrderService → UI/Controller\`: \`PaymentResult(success, transactionId)\`. \`UI/Controller → Buyer\`: \`showSuccess()\`.

\- **\*\*SC-11:\*\*** \`Gateway\` один раз повертає \`PaymentResult(result, transactionId)\` перед \`alt [result = declined] / else [result = success]\`. При відмові \`PaymentService → DB\`: \`savePayment(orderId, declined, transactionId, amount)\`; DB повертає \`paymentSaved\`. Замовлення залишається \`new\`. \`PaymentService → OrderService\` повертає \`PaymentResult(declined, transactionId)\`, потім \`OrderService → UI/Controller\`: \`PaymentResult(declined, transactionId)\`, а \`UI/Controller → Buyer\`: \`showDeclined()\`. При успіху повністю повторюється послідовність SC-10, включно з поверненням результату від \`OrderService\` до \`UI/Controller\` перед \`showSuccess()\`.

\- **\*\*SC-12:\*\*** після \`chargeOrder\` запускається відлік 30 секунд. Після закінчення цього часу \`PaymentService\` викликає \`checkTimeout()\`, який визначає, чи надійшла відповідь від \`Gateway\`. Використовується \`alt [timeout] / else [response received]\`.

  При тайм-ауті відповідь від \`Gateway\` відсутня. \`PaymentService → DB\`: \`savePayment(orderId, pending_verification, NULL, amount)\`; DB повертає \`paymentSaved\`. Замовлення залишається \`new\`. \`PaymentService → OrderService\` повертає \`PaymentResult(pending_verification, NULL)\`, потім \`OrderService → UI/Controller\`: \`PaymentResult(pending_verification, NULL)\`, а \`UI/Controller → Buyer\`: \`showVerificationNotice()\`.

  При отриманні відповіді \`Gateway\` повертає \`PaymentResult(result, transactionId)\`. Вкладений \`alt\` розрізняє \`success\` і \`declined\` та повністю повторює відповідну послідовність SC-10 або SC-11, включно з поверненням результату до \`UI/Controller\` перед викликом відповідного \`showX()\`.

**## 5. Критерії прийняття**

1\. Фрагменти: SC-09 — \`opt\` + \`loop\`; SC-10 — \`opt\`; SC-11 — \`opt\` + \`alt\`; SC-12 — \`opt\` + зовнішній і вкладений \`alt\`.

2\. Return-повідомлення малювати пунктиром: \`order\`, \`orderItems\`, \`PaymentResult(...)\` від \`Gateway\` до \`PaymentService\`, від \`PaymentService\` до \`OrderService\` і від \`OrderService\` до \`UI/Controller\`, \`paymentSaved\`, \`status(paid)\`. У SC-11 шлюз повертає результат лише один раз перед \`alt\`.

3\. Activation bars відповідають викликам і поверненням. \`UI/Controller\` та \`OrderService\` активні до \`opt\` і після нього. \`PaymentService\` не деактивується всередині \`alt\`. У SC-12 \`Gateway\` активується під час \`chargeOrder\` і деактивується один раз після завершення зовнішнього \`alt\`. У гілці \`[timeout]\` повідомлення-відповіді від \`Gateway\` немає.

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
\- Поза обсягом: внутрішня логіка шлюзу, банківські перевірки, повторні спроби, пізні callback-и та реалізація UI.
\- У SC-12 \`Gateway\` активується під час \`chargeOrder\` і деактивується один раз після зовнішнього \`alt\`, а не окремо в кожній його гілці.
\- Те, що замовлення залишається \`new\` при відмові й тайм-ауті, показується відсутністю виклику \`updateOrderStatus\`; окреме повідомлення \`status(new)\` не малюється.\
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
&#x20;@FR-05 @FR-04\
&#x20; Сценарій: SC-09 Надсилання запиту на оплату\
&#x20;   Дано існує CustomerOrder зі статусом "new"\
&#x20;   І загальна сума замовлення дорівнює 500.00\
&#x20;   Коли Покупець ініціює оплату\
&#x20;   Тоді система надсилає платіжному шлюзу ідентифікатор замовлення\
&#x20;   І передає суму 500.00\
\
&#x20; @FR-06\
&#x20; Сценарій: SC-10 Успішна оплата\
&#x20;   Дано існує CustomerOrder зі статусом "new"\
&#x20;   Коли платіжний шлюз підтверджує успішну оплату\
&#x20;   Тоді система змінює CustomerOrder.status на "paid"\
\
&#x20; @FR-07\
&#x20; Сценарій: SC-11 Відмова в оплаті\
&#x20;   Дано існує CustomerOrder зі статусом "new"\
&#x20;   Коли платіжний шлюз відхиляє оплату\
&#x20;   Тоді система залишає CustomerOrder.status рівним "new"\
&#x20;   І повідомляє Покупця про відмову в оплаті\
\
&#x20; @FR-07 @FR-05\
&#x20; Сценарій: SC-12 Таймаут платіжного шлюзу\
&#x20;   Дано існує CustomerOrder зі статусом "new"\
&#x20;   Коли платіжний шлюз не повертає результат протягом 30 секунд\
&#x20;   Тоді система залишає CustomerOrder.status рівним "new"\
&#x20;   І позначає платіж як такий, що потребує перевірки\
&#x20;   І повідомляє Покупця про необхідність перевірки платежу\
&#x20;   І не повідомляє про успішну оплату без підтвердження\
\
Завдання:\
1\. Створи чотири окремі діаграми Mermaid sequenceDiagram: SC-09, SC-10, SC-11, SC-12. Кожна в окремому блоці \`\`\`mermaid.\
2\. Учасників, назви викликів, параметри, return-повідомлення, фрагменти (opt, loop, alt) і умови бери лише зі spec. Нічого не вигадуй і не додавай викликів, яких там немає. Return-повідомлення пунктирними стрілками (-->>). Activation bars через activate/deactivate.\
3\. Mermaid читає діаграму послідовно: кожен activate має рівно один deactivate на кожному шляху виконання, не став deactivate окремо в кожній гілці alt для того самого activate. Діаграма має рендеритися без помилок.\
4\. Після діаграм виведи таблицю самоперевірки: критерій 1–8 зі spec / виконано (так/ні) / що не збігається.\
5\. Не змінюй і не переписуй spec. Якщо знайдеш у ньому суперечність або неясність, не розв'язуй її сама, а перелічи окремо в кінці.

## Відповідь GPT (оригінал)
Нижче — чотири окремі Mermaid sequenceDiagram, побудовані за оновленою специфікацією. У SC-10–SC-12 враховано повернення результату від OrderService до UI/Controller; у SC-09 активація Gateway залишається відкритою до завершення діаграми, а в SC-12 Gateway деактивується один раз після зовнішнього alt.

# SC-09 — Надсилання запиту на оплату

sequenceDiagram
    participant Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant DB
    participant GW as Gateway

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

        loop each order item
            OS->>OS: calculateAmount()
        end

        OS->>PS: processPayment(orderId, amount)
        activate PS
        PS->>GW: chargeOrder(orderId, amount)
        activate GW
        Note over GW: Відповідь у SC-09 не передбачена
        deactivate PS
    end

    deactivate OS
    deactivate UI
    deactivate Buyer

# SC-10 — Успішна оплата

sequenceDiagram
    participant Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant DB
    participant GW as Gateway

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

        loop Обчислення суми позицій
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

        OS-->>UI: PaymentResult(success, transactionId)
        UI->>Buyer: showSuccess()
    end

    deactivate OS
    deactivate UI
    deactivate Buyer

# SC-11 — Відмова в оплаті

sequenceDiagram
    participant Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant DB
    participant GW as Gateway

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

        loop Обчислення суми позицій
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
            OS-->>UI: PaymentResult(declined, transactionId)
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

            OS-->>UI: PaymentResult(success, transactionId)
            UI->>Buyer: showSuccess()
        end

        deactivate PS
    end

    deactivate OS
    deactivate UI
    deactivate Buyer

# sequenceDiagram
    participant Buyer
    participant UI as UI/Controller
    participant OS as OrderService
    participant PS as PaymentService
    participant DB
    participant GW as Gateway

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

        loop Обчислення суми позицій
            OS->>OS: calculateAmount()
        end

        OS->>PS: processPayment(orderId, amount)
        activate PS
        PS->>GW: chargeOrder(orderId, amount)
        activate GW

        Note over PS,GW: Відлік 30 секунд після chargeOrder

        PS->>PS: checkTimeout()

        alt timeout
            PS->>DB: savePayment(orderId, pending_verification, NULL, amount)
            activate DB
            DB-->>PS: paymentSaved
            deactivate DB

            PS-->>OS: PaymentResult(pending_verification, NULL)
            OS-->>UI: PaymentResult(pending_verification, NULL)
            UI->>Buyer: showVerificationNotice()
        else response received
            GW-->>PS: PaymentResult(result, transactionId)

            alt result = success
                PS->>DB: savePayment(orderId, success, transactionId, amount)
                activate DB
                DB-->>PS: paymentSaved
                deactivate DB

                PS-->>OS: PaymentResult(success, transactionId)
                OS->>DB: updateOrderStatus(orderId, paid)
                activate DB
                DB-->>OS: status(paid)
                deactivate DB

                OS-->>UI: PaymentResult(success, transactionId)
                UI->>Buyer: showSuccess()
            else result = declined
                PS->>DB: savePayment(orderId, declined, transactionId, amount)
                activate DB
                DB-->>PS: paymentSaved
                deactivate DB

                PS-->>OS: PaymentResult(declined, transactionId)
                OS-->>UI: PaymentResult(declined, transactionId)
                UI->>Buyer: showDeclined()
            end
        end

        deactivate GW
        deactivate PS
    end

    deactivate OS
    deactivate UI
    deactivate Buyer

Точкова правка до SC-09: щоб кожен `activate` мав відповідний `deactivate`, додай `deactivate GW` останнім рядком діаграми — після `deactivate Buyer`. Так активація `Gateway` триватиме до завершення SC-09, як вимагає специфікація.

## Таблиця самоперевірки

| № | Критерій зі spec                                    | Виконано                | Що не збігається                                                                                                                                                             |
| - | --------------------------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Фрагменти `opt`, `loop`, `alt`                      | Так                     | SC-09: `opt` + `loop`; SC-10: `opt`; SC-11: `opt` + `alt`; SC-12: `opt` + зовнішній і вкладений `alt`.                                                                       |
| 2 | Return-повідомлення пунктиром                       | Так                     | `order`, `orderItems`, усі `PaymentResult(...)`, `paymentSaved`, `status(paid)` показані пунктирними стрілками. У SC-11 шлюз повертає результат один раз перед `alt`.        |
| 3 | Activation bars відповідають викликам і поверненням | Так, після правки SC-09 | `deactivate GW` потрібно додати останнім рядком SC-09. У SC-12 `Gateway` деактивується один раз після зовнішнього `alt`; `PaymentService` — після `alt`, не всередині нього. |
| 4 | Назви викликів і параметри відповідають spec        | Так                     | Використано назви та параметри зі специфікації, зокрема `checkTimeout()`.                                                                                                    |
| 5 | Учасники узгоджені з ER                             | Так                     | `Customer` представлений через `Buyer`; `UI/Controller` і `Gateway` — технічний компонент та зовнішня система, а не сутності ER.                                             |
| 6 | Відповідність Gherkin і spec, без нових правил      | Так                     | Не додано `updateOrderStatus` для відмови чи тайм-ауту. Успішні альтернативи SC-11 і SC-12 показані відповідно до spec.                                                      |
| 7 | Завершальні стани відповідають сценаріям            | Так                     | SC-10: `success`, замовлення `paid`; SC-11: `declined`, замовлення `new`; SC-12: `pending_verification`, замовлення `new`.                                                   |
| 8 | `orderId` у записах і сума відповідає позиціям      | Так                     | `orderId` присутній у `savePayment(...)` та `updateOrderStatus(...)`; сума обчислюється через `calculateAmount()`.                                                           |

## Неясності та зауваження

1. SC-09: `Gateway` не повертає результат. Його активація закривається останнім рядком діаграми лише для дотримання вимоги про парні `activate/deactivate`; це не означає, що шлюз надіслав відповідь.
2. SC-12: `checkTimeout()` показаний після примітки про 30 секунд, як визначено в spec. Сам механізм відліку не деталізується.
3. Сума: у викликах використано параметр `amount`, а не жорстко задане `500.00`, оскільки саме така форма виклику наведена у spec. Передумова SC-09 визначає, що сума дорівнює `500.00`.
4. SC-11: гілка успіху виходить за межі основного результату Gherkin, але явно передбачена специфікацією як альтернативний потік.

Окрім зазначеної точкової правки SC-09, діаграми відповідають наведеним критеріям.

## Мій фідбек (написано мною)

1. Перевірила рендер усіх чотирьох діаграм: `<рендеряться без помилок / вказати, що саме не так>`.
2. Порівняла діаграми з критеріями 1–8 зі spec. Три розбіжності з першої генерації усунені: подвійний `deactivate GW` у SC-12, відсутнє повернення результату до `UI/Controller` і проблема з активацією `Gateway` у SC-09.
3. GPT запропонував додати `deactivate GW` у кінці SC-09. Не прийняла цю пропозицію, оскільки spec вимагає, щоб `Gateway` залишався активним до завершення діаграми.
4. Дрібниці — підписи `loop` різними мовами, тип учасника `Buyer` і момент виклику `checkTimeout()` — свідомо не змінювала, оскільки вони не впливають на основну логіку сценаріїв. Водночас момент виклику `checkTimeout()` може впливати на точність відображення часової послідовності, тому його можна уточнити в spec за потреби.