# Prompt 5 — виправлення підписів діаграми 

## Мій промпт 
Перегенеруй діаграму станів Mermaid (stateDiagram-v2) за оновленим spec\
нижче. Змінено критерій 9 і примітку про deliveryResult. Попередня\
діаграма мала розбіжності: (1) перехід PendingPayment --> paymentResult\
без підпису; (2) підписи «статус new» не вказують, чий це статус\
(Payment чи CustomerOrder). Видай блок \`\`\`mermaid і самоперевірку за\
критеріями 1-9 (по пунктах: виконано / не виконано і чому). Не змінюй spec.\
\
&#x20;Lab4/spec.md знову оновленооо **# Spec — Завдання 4: Процес оплати замовлення (машина станів)**

**## 1. Намір**
Метою роботи є відобразити процес оплати замовлення в інтернет-магазині книжок машиною станів Mermaid. Процес охоплює дві сутності: Payment і CustomerOrder. Стани відображають етапи процесу й результати платежу, а переходи можуть змінювати CustomerOrder.status. Діаграма деталізує UC-06 «Оплата замовлення» (з UC-07 і UC-08) із Завдання 2 та узгоджується з Sequence SC-09–SC-12 із Завдання 3. Машину станів обрано замість Activity, бо вона краще показує повноту переходів і досяжність станів.

**## 2. Стани й переходи**

Ролі: Покупець, OrderService, PaymentService, Платіжний шлюз.

\| Стан | Що описує | Опис |
\|------|-----------|------|
\| New | CustomerOrder (\`status = new\`) | Замовлення створено, оплату не розпочато |
\| PendingPayment | Етап процесу (запису Payment ще немає) | Запит надіслано шлюзу, очікуємо відповідь до 30 с |
\| Paid | Payment (\`success\`) і CustomerOrder (\`status → paid\`) | Шлюз підтвердив платіж |
\| Declined | Payment (\`declined\`); CustomerOrder лишається \`new\` | Шлюз відхилив платіж. Кінцевий стан |
\| PendingVerification | Payment (\`pending_verification\`, \`transaction_id = NULL\`); CustomerOrder лишається \`new\` | Відповіді немає за 30 с. Кінцевий стан |
\| NotificationSent | Етап процесу | Покупця повідомлено про успішну оплату (кінець гілки A) |
\| DeliveryHandled | Етап процесу (CustomerOrder: \`shipped\` або лишається \`paid\`) | Служба доставки відповіла: підтвердила або відмовила (кінець гілки B) |

\| Перехід | Умова / подія | Джерело |
\|---------|---------------|---------|
\| [\*] → New | Замовлення оформлено | UC-04 |
\| New → PendingPayment | Покупець натиснув «Оплатити» і \`CustomerOrder.status = new\` | SC-09 |
\| PendingPayment → Paid | Шлюз повернув success; зберігається Payment, \`status → paid\` | SC-10 |
\| PendingPayment → Declined | Шлюз повернув declined; зберігається Payment, \`status\` лишається \`new\` | SC-11 |
\| PendingPayment → PendingVerification | Таймаут 30 с від \`chargeOrder\`; зберігається Payment, \`status\` лишається \`new\` | SC-12 |
\| Declined → [\*] | Кінцевий стан | SC-11 |
\| PendingVerification → [\*] | Кінцевий стан | SC-12 |
\| Paid → fork | Дві паралельні гілки | SC-10, FR-08 |
\| fork → NotificationSent | Повідомити Покупця про успішну оплату | SC-10 |
\| fork → DeliveryHandled | Передати замовлення службі доставки. Підтверджено: \`status → shipped\`; відмова: \`status\` лишається \`paid\`, повідомлення про помилку. Обидва результати ведуть в один стан DeliveryHandled | SC-13, SC-14 |
\| NotificationSent → join; DeliveryHandled → join | Join чекає обидва стани | FR-08 |
\| join → [\*] | Обидві гілки завершені | FR-08 |

Примітки:
\- Повідомлення гілки A означає лише успішну оплату, а не доставку.
\- Відмова доставки не змінює Payment і не блокує join.
\- Повторна оплата після Declined — новий запуск процесу з New, а не перехід.
\- Якщо \`status ≠ new\`, оплату не ініціюють (переходу немає).
\- Результат доставки (підтверджено або відмова) показується розгалуженням \`deliveryResult\` перед станом \`DeliveryHandled\`; це службовий вузол, а не стан.

**## 3. Критерії прийняття**
1\. Кожен стан досяжний із початкового.
2\. З кожного стану є вихід або він кінцевий (кінцеві: Declined, PendingVerification, [\*] після join).
3\. Результат шлюзу має взаємовиключні гілки, що покривають усі випадки: success / declined / timeout.
4\. Є щонайменше одне розгалуження і одна паралельна гілка з join.
5\. New і Paid відповідають CustomerOrder.status (new, paid) з ER (Завдання 1). Paid, Declined і PendingVerification відповідають статусам платежу (success, declined, pending_verification) зі Sequence (Завдання 3).
6\. Кожен перехід простежується до Sequence (SC-09–SC-12) або до Gherkin (SC-13, SC-14); у підписі кожного переходу діаграми вказано посилання на SC, UC або FR.
7\. Join чекає обидві гілки (NotificationSent і DeliveryHandled); відмова служби доставки не блокує join і не змінює Payment.
8\. Кожна паралельна гілка входить у join одним потоком: взаємовиключні результати гілки зливаються в один стан до join.
9\. Діаграма рендериться без помилок у Mermaid (GitHub). Підписи переходів короткі й не містять символів \`;\` та \`:\` (крім роздільника «стан --> стан : підпис»); частини підпису розділяються комою або тире. Якщо підпис згадує статус, указано чий: Payment або CustomerOrder. Кожен перехід, зокрема до і з \`choice\`, має підпис із посиланням на SC, UC або FR.

**## 4. Поза обсягом**
1\. Повернення коштів і скасування замовлення (UC-10).
2\. Ручна перевірка в PendingVerification (лише фіксуємо стан).
3\. Пізня відповідь шлюзу після таймауту та результати шлюзу, відмінні від success і declined.
4\. Повторна оплата при pending_verification і кількість спроб.
5\. Повторне передавання замовлення службі доставки та подальший життєвий цикл Shipment.

**## 5. Припущення**
\- ER (Завдання 1) не містить сутності Payment: оплата там поза обсягом. Назви й значення статусів платежу взято зі Sequence. Узгодження ER із Payment винесено за межі Завдання 4.
Додано: Lab4/spec.md (версія з коміту 0f5acac).

## Відповідь AI 
stateDiagram-v2
    direction LR

    [*] --> New : Замовлення оформлено UC-04
    New --> PendingPayment : Натиснуто Оплатити, CustomerOrder.status = new SC-09

    state paymentResult <<choice>>
    PendingPayment --> paymentResult : Результат шлюзу або таймаут SC-10, SC-11, SC-12

    paymentResult --> Paid : success, Payment.status = success, CustomerOrder.status = paid SC-10
    paymentResult --> Declined : declined, Payment.status = declined, CustomerOrder.status = new SC-11
    paymentResult --> PendingVerification : timeout 30 с, Payment.status = pending_verification, CustomerOrder.status = new SC-12

    Declined --> [*] : Кінцевий стан SC-11
    PendingVerification --> [*] : Кінцевий стан SC-12

    state forkPoint <<fork>>
    Paid --> forkPoint : Запуск двох паралельних гілок SC-10, FR-08

    forkPoint --> NotificationSent : Повідомити про успішну оплату SC-10
    forkPoint --> deliveryResult : Передати замовлення службі доставки SC-13, SC-14

    state deliveryResult <<choice>>
    deliveryResult --> DeliveryHandled : Підтверджено, CustomerOrder.status = shipped SC-13
    deliveryResult --> DeliveryHandled : Відмова, CustomerOrder.status = paid SC-14

    state joinPoint <<join>>
    NotificationSent --> joinPoint : Гілка A завершена FR-08
    DeliveryHandled --> joinPoint : Гілка B завершена FR-08

    joinPoint --> [*] : Обидві гілки завершені FR-08

### Самоперевірка за критеріями 1–9

1. Виконано. Усі сім станів досяжні з початкового стану: `New`, `PendingPayment`, `Paid`, `Declined`, `PendingVerification`, `NotificationSent` і `DeliveryHandled`.
2. Виконано. `New`, `PendingPayment` і `Paid` мають виходи. `Declined` і `PendingVerification` — кінцеві. `NotificationSent` і `DeliveryHandled` ведуть до join, після якого процес завершується.
3. Виконано. Після `PendingPayment` через `paymentResult` передбачено три взаємовиключні результати: `success`, `declined` і `timeout 30 с`.
4. Виконано. Є розгалуження `<<choice>>` для результатів шлюзу та паралельне розгалуження `<<fork>>` після `Paid` із подальшим `<<join>>`.
5. Виконано відповідно до spec. `New` відповідає `CustomerOrder.status = new`, `Paid` — `CustomerOrder.status = paid`. `Paid`, `Declined` і `PendingVerification` також відповідають статусам `Payment` — `success`, `declined` і `pending_verification`. Зауваження: згідно з §5 сутності `Payment` у ER немає, тому статуси платежу звіряються із Sequence.
6. Виконано. Кожен перехід має посилання на джерело SC, UC або FR. Зокрема, підписано перехід `PendingPayment --> paymentResult`, а також переходи до та з `choice`, `fork` і `join`.
7. Виконано. Join очікує обидві гілки — `NotificationSent` і `DeliveryHandled`. Відмова доставки залишає `CustomerOrder.status = paid`, не змінює `Payment` і не перешкоджає завершенню гілки.
8. Виконано. Два взаємовиключні результати доставки сходяться в одному стані `DeliveryHandled`. Від нього до join веде лише один потік. Гілка повідомлення також входить у join одним потоком.
9. Виконано за перевіркою тексту та структури. Використано `stateDiagram-v2`. Підписи короткі, без `;` і додаткових `:`. Частини розділено комами. У кожному підписі зі статусом зазначено власника — `Payment` або `CustomerOrder`. Усі переходи, включно з переходами до та з `choice`, мають посилання на SC, UC або FR. Фактичний рендер у GitHub у цій відповіді не запускався, тому остаточне відображення потрібно перевірити в GitHub.

Примітка: `paymentResult` і `deliveryResult` — службові вузли `<<choice>>`, а не додаткові стани процесу. `deliveryResult` використано саме перед `DeliveryHandled`, як зазначено в оновленому spec.

## Результат перевірки 
## Результат перевірки (написано самостійно)
Перевірила діаграму за критеріями 1-9. Критерії 1-8 виконано: усі стани досяжні, `choice` має три взаємовиключні гілки, `join` має два входи, усі переходи підписані з посиланням на SC/UC/FR, у підписах статусів указано сутність (Payment або CustomerOrder). Підписи не містять `;` і зайвих `:`, діаграма рендериться.

Знайдена розбіжність зі spec: службовий вузол `deliveryResult` оголошено після першого використання, тому Mermaid намалював його як звичайний стан-прямокутник, а не як `choice`. Для `paymentResult`, оголошеного до використання, усе відображається правильно. Spec називає `deliveryResult` службовим вузлом, а не станом. Виправлення: додаю до критерію 9 вимогу оголошувати службові вузли до першого використання і перегенеровую діаграму (prompt 6).