# Lab 4 — процес оплати (машина станів)

```mermaid
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
```