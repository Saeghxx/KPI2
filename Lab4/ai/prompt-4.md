# Prompt 4 — виправлення синтаксису діаграми 

## Мій промпт 
Діаграма не рендериться: «Діаграма некоректна або не підтримується\
stateDiagram-v2». Імовірна причина: символи ; і : у підписах переходів.\
У spec додано критерій 9 (див. нижче). Перегенеруй діаграму за оновленим\
spec: підписи короткі, без ; і :, частини розділяй комою або тире,\
посилання на SC/UC/FR лиши. Видай блок \`\`\`mermaid і самоперевірку за\
критеріями 1-9. Не змінюй spec.\
\
&#x20;Lab4/spec.md оновлено **# Spec — Завдання 4: Процес оплати замовлення (машина станів)**

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

**## 3. Критерії прийняття**
1\. Кожен стан досяжний із початкового.
2\. З кожного стану є вихід або він кінцевий (кінцеві: Declined, PendingVerification, [\*] після join).
3\. Результат шлюзу має взаємовиключні гілки, що покривають усі випадки: success / declined / timeout.
4\. Є щонайменше одне розгалуження і одна паралельна гілка з join.
5\. New і Paid відповідають CustomerOrder.status (new, paid) з ER (Завдання 1). Paid, Declined і PendingVerification відповідають статусам платежу (success, declined, pending_verification) зі Sequence (Завдання 3).
6\. Кожен перехід простежується до Sequence (SC-09–SC-12) або до Gherkin (SC-13, SC-14); у підписі кожного переходу діаграми вказано посилання на SC, UC або FR.
7\. Join чекає обидві гілки (NotificationSent і DeliveryHandled); відмова служби доставки не блокує join і не змінює Payment.
8\. Кожна паралельна гілка входить у join одним потоком: взаємовиключні результати гілки зливаються в один стан до join.
9\. Діаграма рендериться без помилок у Mermaid (GitHub). Підписи переходів короткі й не містять символів \`;\` та \`:\` (крім роздільника «стан --> стан : підпис»); частини підпису розділяються комою або тире.

**## 4. Поза обсягом**
1\. Повернення коштів і скасування замовлення (UC-10).
2\. Ручна перевірка в PendingVerification (лише фіксуємо стан).
3\. Пізня відповідь шлюзу після таймауту та результати шлюзу, відмінні від success і declined.
4\. Повторна оплата при pending_verification і кількість спроб.
5\. Повторне передавання замовлення службі доставки та подальший життєвий цикл Shipment.

**## 5. Припущення**
\- ER (Завдання 1) не містить сутності Payment: оплата там поза обсягом. Назви й значення статусів платежу взято зі Sequence. Узгодження ER із Payment винесено за межі Завдання 4.

Додано: Lab4/spec.md (версія з коміту 7a7d9af)

## Відповідь AI
stateDiagram-v2
    direction LR

    [*] --> New : Оформлення замовлення UC-04
    New --> PendingPayment : Оплата, status = new SC-09

    state paymentResult <<choice>>
    PendingPayment --> paymentResult
    paymentResult --> Paid : success, збережено Payment SC-10
    paymentResult --> Declined : declined, статус new SC-11
    paymentResult --> PendingVerification : timeout 30 с, статус new SC-12

    Declined --> [*] : Кінцевий стан SC-11
    PendingVerification --> [*] : Кінцевий стан SC-12

    state forkPoint <<fork>>
    Paid --> forkPoint : Паралельні гілки SC-10, FR-08

    forkPoint --> NotificationSent : Повідомити про успішну оплату SC-10
    forkPoint --> deliveryResult : Передати замовлення на доставку SC-13, SC-14

    state deliveryResult <<choice>>
    deliveryResult --> DeliveryHandled : Підтверджено, status = shipped SC-13
    deliveryResult --> DeliveryHandled : Відмова, status лишається paid SC-14

    state joinPoint <<join>>
    NotificationSent --> joinPoint : Гілка A завершена FR-08
    DeliveryHandled --> joinPoint : Гілка B завершена FR-08

    joinPoint --> [*] : Обидві гілки завершені FR-08

### Самоперевірка за критеріями 1–9

1. Виконано. Усі сім станів досяжні з початкового стану: `New`, `PendingPayment`, три результати оплати, `NotificationSent` і `DeliveryHandled`.
2. Виконано. `New`, `PendingPayment` і `Paid` мають виходи; `Declined` та `PendingVerification` — кінцеві; `NotificationSent` і `DeliveryHandled` ведуть до join, після якого процес завершується.
3. Виконано. Після `PendingPayment` choice розділяє три взаємовиключні результати: `success`, `declined` і `timeout 30 с`.
4. Виконано. Є `<<choice>>` для результатів шлюзу, `<<fork>>` для двох паралельних гілок після `Paid` і `<<join>>` для їх об'єднання.
5. Виконано відповідно до spec. `New` відповідає `CustomerOrder.status = new`; `Paid` — `CustomerOrder.status = paid` і `Payment.success`. `Declined` та `PendingVerification` відповідають статусам платежу зі Sequence. Зауваження: сам spec зазначає, що сутності `Payment` у ER немає.
6. Виконано. Кожен перехід має посилання на джерело: UC-04, SC-09–SC-14 або FR-08.
7. Виконано. До join входять два потоки — від `NotificationSent` і `DeliveryHandled`. Відмова доставки приводить до `DeliveryHandled`, не змінює `Payment` і не блокує join.
8. Виконано. Підтвердження та відмова доставки — взаємовиключні переходи, які сходяться в одному стані `DeliveryHandled` до join. Отже, гілка доставки входить у join одним потоком.
9. Виконано за синтаксисом діаграми. Використано `stateDiagram-v2`, короткі підписи без `;` і `:` всередині тексту підписів. Частини підписів розділено комами. Синтаксис підготовлено для Mermaid у GitHub, але фактичний рендер у GitHub у цій відповіді не запускався.

### Припущення

- `deliveryResult` — допоміжний псевдостан `<<choice>>`, потрібний, щоб два взаємовиключні результати доставки сходилися в `DeliveryHandled` до join.
- Умови `success`, `declined` і `timeout 30 с` трактуються як взаємовиключні результати очікування відповіді шлюзу.
- `status = new` та `status = paid` у підписах стосуються `CustomerOrder`, як визначено в таблиці станів.
- Посилання `FR-08` використано для переходів до join і завершення процесу відповідно до таблиці переходів у spec.