# Prompt 6 — порядок оголошення службових вузлів

## Мій промпт 
Перегенеруй діаграму станів Mermaid (stateDiagram-v2) за оновленим spec
нижче. У критерії 9 додано вимогу оголошувати службові вузли (choice,
fork, join) до першого використання. У попередній версії рядок
state deliveryResult <<choice>> стояв після першого переходу до нього,
тому Mermaid намалював його як звичайний стан, а не як службовий вузол.
Видай блок ```mermaid і самоперевірку за критеріями 1-9 (по пунктах:
виконано / не виконано і чому). Не змінюй spec.

Додано: Lab4/spec.md (версія з коміту 1fda890).

## Відповідь AI 
stateDiagram-v2
    direction LR

    state paymentResult <<choice>>
    state forkPoint <<fork>>
    state deliveryResult <<choice>>
    state joinPoint <<join>>

    [*] --> New : Замовлення оформлено UC-04
    New --> PendingPayment : Натиснуто Оплатити, CustomerOrder.status = new SC-09

    PendingPayment --> paymentResult : Результат шлюзу або таймаут SC-10, SC-11, SC-12

    paymentResult --> Paid : success, Payment.status = success, CustomerOrder.status = paid SC-10
    paymentResult --> Declined : declined, Payment.status = declined, CustomerOrder.status = new SC-11
    paymentResult --> PendingVerification : timeout 30 с, Payment.status = pending_verification, CustomerOrder.status = new SC-12

    Declined --> [*] : Кінцевий стан SC-11
    PendingVerification --> [*] : Кінцевий стан SC-12

    Paid --> forkPoint : Дві паралельні гілки SC-10, FR-08
    forkPoint --> NotificationSent : Повідомити про успішну оплату SC-10
    forkPoint --> deliveryResult : Передати замовлення на доставку SC-13, SC-14

    deliveryResult --> DeliveryHandled : Підтверджено, CustomerOrder.status = shipped SC-13
    deliveryResult --> DeliveryHandled : Відмова, CustomerOrder.status = paid SC-14

    NotificationSent --> joinPoint : Гілка A завершена FR-08
    DeliveryHandled --> joinPoint : Гілка B завершена FR-08
    joinPoint --> [*] : Обидві гілки завершені FR-08

### Самоперевірка за критеріями 1–9

1. Виконано. Усі сім станів досяжні з початкового стану: `New`, `PendingPayment`, `Paid`, `Declined`, `PendingVerification`, `NotificationSent` і `DeliveryHandled`.
2. Виконано. `New`, `PendingPayment` і `Paid` мають виходи. `Declined` і `PendingVerification` — кінцеві. `NotificationSent` і `DeliveryHandled` ведуть до join, після якого процес завершується.
3. Виконано. Після `PendingPayment` через `paymentResult` передбачено три взаємовиключні результати: `success`, `declined` і `timeout 30 с`.
4. Виконано. Є `<<choice>>` для результатів шлюзу, `<<fork>>` для двох паралельних гілок після `Paid` і `<<join>>` для їх синхронізації.
5. Виконано відповідно до spec. `New` відповідає `CustomerOrder.status = new`, `Paid` — `CustomerOrder.status = paid`. Стани `Paid`, `Declined` і `PendingVerification` також відповідають статусам `Payment` — `success`, `declined` і `pending_verification`. У §5 зазначено, що сутності `Payment` у ER немає, тому статуси платежу звіряються зі Sequence.
6. Виконано. Кожен перехід має посилання на SC, UC або FR, зокрема переходи до та з усіх службових вузлів.
7. Виконано. Join очікує обидві гілки — `NotificationSent` і `DeliveryHandled`. Відмова доставки залишає `CustomerOrder.status = paid`, не змінює `Payment` і не блокує join.
8. Виконано. Підтвердження та відмова доставки сходяться в одному стані `DeliveryHandled` до join. Тому гілка доставки входить у join одним потоком. Гілка повідомлення також має один вхідний потік до join.
9. Виконано за структурою коду. Усі службові вузли (`paymentResult`, `forkPoint`, `deliveryResult`, `joinPoint`) оголошені до першого використання. Підписи короткі, не містять `;` і додаткових `:`, частини розділено комами. У підписах зі статусами вказано власника — `Payment` або `CustomerOrder`. Кожен перехід має посилання на SC, UC або FR. Фактичний рендер у GitHub у цій відповіді не запускався, тому його потрібно перевірити там.

Припущення: `paymentResult` і `deliveryResult` — службові вузли `<<choice>>`, а не додаткові стани. Результати доставки є взаємовиключними й обидва приводять до `DeliveryHandled`, як зазначено в spec.

## Результат перевірки 
Перевірила нову діаграму за критеріями 1-9. Службові вузли (paymentResult, forkPoint, deliveryResult, joinPoint) оголошено до першого використання, тому deliveryResult тепер відображається як службовий вузол, а не як звичайний стан (розбіжність із prompt 5 усунуто). Join має два входи (NotificationSent і DeliveryHandled), choice після PendingPayment має три взаємовиключні гілки, кожен перехід має посилання на SC, UC або FR, у підписах статусів указано, чий це статус (Payment або CustomerOrder). Підписи не містять `;` і зайвих `:`. Рендер перевірила в GitHub / preview: помилок немає.