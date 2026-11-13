# Spec — Завдання 5: Архітектура (Component Diagram)

## 1. Намір
Показати архітектуру онлайн-книгарні: компоненти, інтерфейси, залежності,
межу системи й зовнішні системи. Модель узгоджена з ER (`Lab2/model/er.md`),
вимогами (Завдання 2), Sequence SC-09–SC-12 (Завдання 3) і станами оплати (Завдання 4).
Інструмент: PlantUML `component` (має provided/required-інтерфейси).

## 2. Стиль
Моноліт із шарами: UI → API → сервіси → Data Access → БД.
Мікросервіси не потрібні: у вимогах немає масштабування чи окремого деплою.

## 3. Компоненти
| Компонент | Що робить | Звідки |
|---|---|---|
| Web UI | Екрани Гостя, Покупця, Адміністратора | UC-01–14 |
| API | Прийом запитів, авторизація, кошик у сесії | `UI/Controller` (Lab3), FR-03 |
| AccountService | Реєстрація, вхід, роль, пароль | FR-02, FR-12; Customer |
| CatalogService | Книги, автори, видавництва, відгуки | FR-01, 10, 11; Book, Author, Publisher, Review |
| OrderService | Замовлення, скасування, статуси | FR-04, 09; CustomerOrder, OrderItem |
| PaymentService | Оплата, таймаут 30 с, запис платежу | FR-05–07; Payment |
| ShippingService | Передача оплаченого замовлення на доставку | FR-08; Shipment |
| Data Access | Єдиний доступ до БД | `DB` (Lab3) |
| БД | Зберігання сутностей ER | ER |

## 4. Зовнішні системи (поза межею)
Платіжний шлюз і Служба доставки (Lab2, Lab3).

## 5. Інтерфейси (provided → хто потребує)
- IAccounts, ICatalog, IOrders: сервіс → API.
- IPayments (`processPayment(orderId, amount)`): PaymentService → OrderService.
- IShipping (`submitShipment(orderId)`): ShippingService → OrderService.
- IPaymentGateway (`chargeOrder(orderId, amount)`): Платіжний шлюз → PaymentService.
- IDeliveryProvider (`createShipment(orderId)`): Служба доставки → ShippingService.
- IRepository (`readOrder`, `readOrderItems`, `savePayment`, `updateOrderStatus`): Data Access → усі сервіси.

## 6. Рішення
1. Кошик живе в сесії API, у БД його немає (рішення Lab2).
2. Вхід належить AccountService; UC входу в Завданні 2 немає, це відома прогалина.
3. Після `paid` доставку запускає OrderService через ShippingService.
4. Залежності односпрямовані, циклів немає.

## 7. Поза обсягом
SMS/Email, OAuth, Category, wishlist, повернення коштів, деплоймент.

## 8. Критерії прийняття
1. Кожен компонент має посилання на FR/UC і сутність ER.
2. Кожен required-інтерфейс має рівно одного provider.
3. Назви операцій збігаються з повідомленнями Lab3.
4. Межа системи намальована, зовнішні системи поза нею.
5. Немає циклічних залежностей.
6. Сервіси звертаються до БД лише через Data Access.
7. Кожна сутність ER (крім Category і wishlist) має компонент-власника.
8. Діаграма читабельна й рендериться без помилок.