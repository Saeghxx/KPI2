# Prompt 2 — генерація діаграми

## Мій промпт 
Згенеруй діаграму станів Mermaid (stateDiagram-v2) для процесу оплати
замовлення за моєю специфікацією нижче.

Вимоги:
1. Використовуй лише стани й переходи зі spec. Нічого не додавай і не прибирай.
2. Назви станів: New, PendingPayment, Paid, Declined, PendingVerification.
3. Розгалуження після PendingPayment зроби через <<choice>>
   (success / declined / timeout), на переходах пиши умови зі spec.
4. Після Paid зроби <<fork>> на дві гілки (повідомлення Покупцю; передавання
   на доставку) і <<join>>, що чекає обидві. Гілка доставки завершується і
   при підтвердженні, і при відмові.
5. Declined і PendingVerification — кінцеві стани (перехід у [*]).
6. Діаграма має читатися без перетину стрілок.
7. Видай лише блок ```mermaid. Після нього окремо перелічи, як ти перевірив
   критерії прийняття 1-7 зі spec (по пунктах: виконано / не виконано і чому)
   та які припущення довелося зробити.

Не змінюй spec і не пропонуй нових вимог.

 Lab4spec.md # Spec — Завдання 4: Процес оплати замовлення (машина станів)

## 1. Намір
Метою роботи є відобразити процес оплати замовлення в інтернет-магазині книжок машиною станів Mermaid. Процес охоплює дві сутності: Payment і CustomerOrder. Стани відображають етапи процесу й результати платежу, а переходи можуть змінювати CustomerOrder.status. Діаграма деталізує UC-06 «Оплата замовлення» (з UC-07 і UC-08) із Завдання 2 та узгоджується з Sequence SC-09–SC-12 із Завдання 3. Машину станів обрано замість Activity, бо вона краще показує повноту переходів і досяжність станів.

## 2. Стани й переходи

Ролі: Покупець, OrderService, PaymentService, Платіжний шлюз.

| Стан | Що описує | Опис |
|------|-----------|------|
| New | CustomerOrder (`status = new`) | Замовлення створено, оплату не розпочато |
| PendingPayment | Етап процесу (запису Payment ще немає) | Запит надіслано шлюзу, очікуємо відповідь до 30 с |
| Paid | Payment (`success`) і CustomerOrder (`status → paid`) | Шлюз підтвердив платіж |
| Declined | Payment (`declined`); CustomerOrder лишається `new` | Шлюз відхилив платіж. Кінцевий стан |
| PendingVerification | Payment (`pending_verification`, `transaction_id = NULL`); CustomerOrder лишається `new` | Відповіді немає за 30 с. Кінцевий стан |

| Перехід | Умова / подія | Джерело |
|---------|---------------|---------|
| [*] → New | Замовлення оформлено | UC-04 |
| New → PendingPayment | Покупець натиснув «Оплатити» і `CustomerOrder.status = new` | SC-09 |
| PendingPayment → Paid | Шлюз повернув success; зберігається Payment, `status → paid` | SC-10 |
| PendingPayment → Declined | Шлюз повернув declined; зберігається Payment, `status` лишається `new` | SC-11 |
| PendingPayment → PendingVerification | Таймаут 30 с від `chargeOrder`; зберігається Payment, `status` лишається `new` | SC-12 |
| Declined → [*] | Кінцевий стан | SC-11 |
| PendingVerification → [*] | Кінцевий стан | SC-12 |
| Paid → fork | Дві паралельні гілки | SC-10, FR-08 |
| Гілка A (fork → join) | Повідомити Покупця про успішну оплату | SC-10 |
| Гілка B (fork → join) | Передати замовлення службі доставки. Підтверджено: `status → shipped`. Відмова: `status` лишається `paid`, повідомлення про помилку; гілка теж завершується | SC-13, SC-14 |
| join → [*] | Обидві гілки завершені | FR-08 |

Примітки:
- Повідомлення гілки A означає лише успішну оплату, а не доставку.
- Відмова доставки не змінює Payment і не блокує join.
- Повторна оплата після Declined — новий запуск процесу з New, а не перехід.
- Якщо `status ≠ new`, оплату не ініціюють (переходу немає).

## 3. Критерії прийняття
1. Кожен стан досяжний із початкового.
2. З кожного стану є вихід або він кінцевий (кінцеві: Declined, PendingVerification, [*] після join).
3. Результат шлюзу має взаємовиключні гілки, що покривають усі випадки: success / declined / timeout.
4. Є щонайменше одне розгалуження і одна паралельна гілка з join.
5. New і Paid відповідають CustomerOrder.status (new, paid) з ER (Завдання 1). Paid, Declined і PendingVerification відповідають статусам платежу (success, declined, pending_verification) зі Sequence (Завдання 3).
6. Кожен перехід простежується до Sequence (SC-09–SC-12) або до Gherkin (SC-13, SC-14).
7. Join чекає обидві гілки; відмова служби доставки не блокує join і не змінює Payment.

## 4. Поза обсягом
1. Повернення коштів і скасування замовлення (UC-10).
2. Ручна перевірка в PendingVerification (лише фіксуємо стан).
3. Пізня відповідь шлюзу після таймауту та результати шлюзу, відмінні від success і declined.
4. Повторна оплата при pending_verification і кількість спроб.
5. Повторне передавання замовлення службі доставки та подальший життєвий цикл Shipment.

## 5. Припущення
- ER (Завдання 1) не містить сутності Payment: оплата там поза обсягом. Назви й значення статусів платежу взято зі Sequence. Узгодження ER із Payment винесено за межі Завдання 4. Додано: Lab4/spec.md (версія з коміту `b94163f`).

## Відповідь AI 
stateDiagram-v2
    direction LR

    [*] --> New : Замовлення оформлено (UC-04)
    New --> PendingPayment : Покупець натиснув «Оплатити»\n[CustomerOrder.status = new]

    state paymentResult <<choice>>
    PendingPayment --> paymentResult

    paymentResult --> Paid : [success]\nЗберегти Payment;\nCustomerOrder.status = paid
    paymentResult --> Declined : [declined]\nЗберегти Payment;\nCustomerOrder.status лишається new
    paymentResult --> PendingVerification : [timeout 30 с від chargeOrder]\nЗберегти Payment;\nCustomerOrder.status лишається new

    Declined --> [*]
    PendingVerification --> [*]

    state forkPoint <<fork>>
    state joinPoint <<join>>

    Paid --> forkPoint
    forkPoint --> joinPoint : Повідомити Покупця про успішну оплату
    forkPoint --> deliveryResult : Передати замовлення службі доставки

    state deliveryResult <<choice>>
    deliveryResult --> joinPoint : [доставку підтверджено]\nCustomerOrder.status = shipped
    deliveryResult --> joinPoint : [служба доставки відмовила]\nCustomerOrder.status лишається paid;\nповідомити про помилку

    joinPoint --> [*]

    \#chatgpt-mermaid-\_r_ck\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_ck\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_ck\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_ck\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_ck\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_ck\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_ck\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_ck\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_ck\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_ck\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_ck\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_ck\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_ck\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_ck\_ p{margin:0;}#chatgpt-mermaid-\_r_ck\_ defs [id$="-barbEnd"]{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_ck\_ g.stateGroup text{fill:rgb(31, 78, 148);stroke:none;font-size:10px;}#chatgpt-mermaid-\_r_ck\_ g.stateGroup text{fill:rgb(237, 237, 237);stroke:none;font-size:10px;}#chatgpt-mermaid-\_r_ck\_ g.stateGroup .state-title{font-weight:bolder;fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ g.stateGroup rect{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_ck\_ g.stateGroup line{stroke:rgb(175, 175, 175);stroke-width:1;}#chatgpt-mermaid-\_r_ck\_ .transition{stroke:rgb(175, 175, 175);stroke-width:1;fill:none;}#chatgpt-mermaid-\_r_ck\_ .stateGroup .composit{fill:transparent;border-bottom:1px;}#chatgpt-mermaid-\_r_ck\_ .stateGroup .alt-composit{fill:#e0e0e0;border-bottom:1px;}#chatgpt-mermaid-\_r_ck\_ .state-note{stroke:rgb(58, 132, 63);fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_ck\_ .state-note text{fill:rgb(237, 237, 237);stroke:none;font-size:10px;}#chatgpt-mermaid-\_r_ck\_ .stateLabel .box{stroke:none;stroke-width:0;fill:rgb(9, 23, 44);opacity:0.5;}#chatgpt-mermaid-\_r_ck\_ .edgeLabel .label rect{fill:rgb(9, 23, 44);opacity:0.5;}#chatgpt-mermaid-\_r_ck\_ .edgeLabel{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-\_r_ck\_ .edgeLabel p{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_ck\_ .edgeLabel rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_ck\_ .edgeLabel .label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ .label div .edgeLabel{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ .stateLabel text{fill:rgb(237, 237, 237);font-size:10px;font-weight:bold;}#chatgpt-mermaid-\_r_ck\_ .node circle.state-start{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_ck\_ .node .fork-join{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_ck\_ .node circle.state-end{fill:rgb(31, 78, 148);stroke:transparent;stroke-width:1.5;}#chatgpt-mermaid-\_r_ck\_ .end-state-inner{fill:transparent;stroke-width:1.5;}#chatgpt-mermaid-\_r_ck\_ .node rect{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_ck\_ .node polygon{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_ck\_ [id$="-barbEnd"]{fill:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_ck\_ .statediagram-cluster rect{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_ck\_ .cluster-label,#chatgpt-mermaid-\_r_ck\_ .nodeLabel{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ .statediagram-cluster rect.outer{rx:5px;ry:5px;}#chatgpt-mermaid-\_r_ck\_ .statediagram-state .divider{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_ck\_ .statediagram-state .title-state{rx:5px;ry:5px;}#chatgpt-mermaid-\_r_ck\_ .statediagram-cluster.statediagram-cluster .inner{fill:transparent;}#chatgpt-mermaid-\_r_ck\_ .statediagram-cluster.statediagram-cluster-alt .inner{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_ck\_ .statediagram-cluster .inner{rx:0;ry:0;}#chatgpt-mermaid-\_r_ck\_ .statediagram-state rect.basic{rx:5px;ry:5px;}#chatgpt-mermaid-\_r_ck\_ .statediagram-state rect.divider{stroke-dasharray:10,10;fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_ck\_ .note-edge{stroke-dasharray:5;}#chatgpt-mermaid-\_r_ck\_ .statediagram-note rect{fill:rgb(48, 48, 48);stroke:rgb(58, 132, 63);stroke-width:1px;rx:0;ry:0;}#chatgpt-mermaid-\_r_ck\_ .statediagram-note rect{fill:rgb(48, 48, 48);stroke:rgb(58, 132, 63);stroke-width:1px;rx:0;ry:0;}#chatgpt-mermaid-\_r_ck\_ .statediagram-note text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ .statediagram-note .nodeLabel{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ .statediagram .edgeLabel{color:red;}#chatgpt-mermaid-\_r_ck\_ [id$="-dependencyStart"],#chatgpt-mermaid-\_r_ck\_ [id$="-dependencyEnd"]{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);stroke-width:1;}#chatgpt-mermaid-\_r_ck\_ .statediagramTitleText{text-anchor:middle;font-size:18px;fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].statediagram-cluster rect{fill:rgb(9, 23, 44);stroke:url(#chatgpt-mermaid-\_r_ck\_-gradient);stroke-width:1;}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].statediagram-cluster rect.outer{rx:5px;ry:5px;filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ck\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_ck\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_ck\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_ck\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_ck\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ck\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_ck\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ck\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}NewPendingPaymentPaid;\nCustomerOrder.status=paidDeclinedлишаєтьсяnewPendingVerificationdeliveryResult;\nповідомитипропомилкуЗамовлення оформлено(UC-04)Покупець натиснув«Оплатити»\n[CustomerOrder.status = new][success]\nЗберегтиPayment[declined]\nЗберегтиPayment[timeout 30 с відchargeOrder]\nЗберегтиPaymentПовідомити Покупця проуспішну оплатуПередати замовленняслужбі доставки[доставкупідтверджено]\nCustomerOrder.status = shipped[служба доставкивідмовила]\nCustomerOrder.status лишається paid

### Перевірка критеріїв прийняття

1. Виконано. Усі п'ять станів досяжні: `New` — від початкового стану, `PendingPayment` — після ініціювання оплати, решта — через відповідні результати шлюзу або таймаут.
2. Виконано. `PendingPayment` має виходи через три результати. `New` має перехід до `PendingPayment`. `Paid` переходить до fork. `Declined` і `PendingVerification` є кінцевими. Після join процес завершується.
3. Виконано. Після `PendingPayment` використано `<<choice>>` із трьома взаємовиключними умовами: `success`, `declined`, `timeout`. Вони відповідають сценаріям SC-10–SC-12.
4. Виконано. Є розгалуження через `<<choice>>` після `PendingPayment` і паралельне розгалуження через `<<fork>>` після `Paid`, яке завершується в `<<join>>`.
5. Виконано за специфікацією. `New` відповідає `CustomerOrder.status = new`. `Paid` відповідає `CustomerOrder.status = paid` і `Payment.status = success`. `Declined` і `PendingVerification` відповідають статусам платежу зі Sequence. Водночас у §5 зазначено, що сутності `Payment` в ER немає, тому перевірка її статусів за ER не виконується.
6. Не виконано повністю за буквальним формулюванням критерію. Переходи оплати простежуються до SC-09–SC-12, гілки доставки — до Gherkin SC-13 і SC-14. Але перехід `[*] → New` має джерело UC-04, а не SC-09–SC-12 чи SC-13–SC-14. Тому він не відповідає переліку джерел у критерії 6.
7. Виконано. `<<join>>` отримує обидві паралельні гілки. Гілка доставки завершується як при підтвердженні, так і при відмові. При відмові `CustomerOrder.status` залишається `paid`, а `Payment` не змінюється.

### Припущення

- Для результатів шлюзу використано умови `[success]`, `[declined]` і `[timeout 30 с від chargeOrder]` відповідно до опису переходів у специфікації.
- Для результату доставки використано окремий `<<choice>>`, щоб показати два взаємовиключні результати однієї гілки, які обидва ведуть до join.
- Повідомлення Покупцю про успішну оплату та передавання замовлення на доставку починаються паралельно після `Paid`.
- `Paid` одночасно відображає успішний стан платежу та зміну статусу замовлення.
- Запис `Payment` створюється після результату шлюзу, а не під час `PendingPayment`, відповідно до опису станів і SC-10–SC-12.