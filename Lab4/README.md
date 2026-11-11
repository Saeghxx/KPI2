# Lab 4 — процес оплати (машина станів)

```mermaid
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
```