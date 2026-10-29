 Data Flow and Behavior
1. مخطط التسلسل لعملية الدفع وإتمام الطلب (Checkout & Stripe Payment Sequence Diagram)
يوضح هذا المخطط تفاعل العميل مع طبقات النظام المختلفة (MVC Views, Services, Stripe API, و SQL Server) عند إتمام عملية الشراء:

sequenceDiagram
    autonumber
    actor Customer as Customer
    participant UI as ASP.NET MVC View
    participant Cart as Cart Service
    participant Stripe as Stripe API
    participant DB as SQL Server

    Customer->>UI: Click Checkout & Pay
    UI->>Cart: Validate Cart Items
    Cart->>Stripe: Create Payment Intent
    Stripe-->>Cart: Webhook Token Success
    Cart->>DB: Save Order & Update Stock
    DB-->>UI: Transaction Confirmed
    UI-->>Customer: Redirect to Success Page
2. مخططات تدفق البيانات (Data Flow Diagrams - DFD)
أ. Context Diagram (Level 0 DFD)
يوضح النظام ككل والأطراف الخارجية المتعاملة معه:

flowchart TD
    Customer[Customer] -->|Order & Payment Data| System[TechXpress System]
    Admin[Admin] -->|Manage Inventory & Products| System
    Delivery[Delivery Agent] -->|Track Shipping Status| System
    System -->|Order Confirmation & Invoice| Customer
    System -->|Process Payment| Stripe[Stripe API]
    Stripe -->|Webhook Response| System
ب. Level 0 DFD (العمليات الكبرى)
العمليات الرئيسية داخل النظام:

flowchart TD
    subgraph TechXpress ["TechXpress Core System"]
        OP1[1. User Identity & Auth]
        OP2[2. Catalog Search & Filter]
        OP3[3. Shopping Cart Management]
        OP4[4. Checkout & Order Processing]
    end

    C[Customer] --> OP1
    C --> OP2
    C --> OP3
    C --> OP4
    OP4 --> DB[(SQL Server DB)]
    OP4 --> Stripe[Stripe API]
ج. Level 1 DFD (تفاصيل عملية إتمام الطلب والدفع)
تفكيك العملية رقم 4 (Checkout & Order Processing) إلى خطوات فرعية:

flowchart TD
    subgraph Checkout_Process ["4. Checkout & Order Sub-processes"]
        P41[4.1 Validate Cart & Stock]
        P42[4.2 Calculate Total & Taxes]
        P43[4.3 Process Stripe Payment]
        P44[4.4 Deduct Inventory]
        P45[4.5 Create Order Record]
    end

    User[Customer] -->|Submit Cart| P41
    P41 -->|Cart Items| P42
    P42 -->|Amount| P43
    P43 <-->|API Token| PaymentGateway[Stripe API]
    P43 -->|Success| P44
    P44 -->|Update Stock| ProductDB[(Products Table)]
    P43 -->|Success| P45
    P45 -->|Save Order| OrderDB[(Orders Table)]
3. مخطط النشاط لعملية الشراء (Activity Diagram)
يوضح مسار اتخاذ القرار لرحلة المستخدم خطوة بخطوة:

flowchart TD
    Start([Start]) --> Browse[Browse & Search Products]
    Browse --> AddCart[Add Items to Cart]
    AddCart --> Checkout{Is User Logged In?}
    Checkout -->|No| Login[Login via ASP.NET Identity]
    Login --> Shipping[Enter Shipping Details]
    Checkout -->|Yes| Shipping
    Shipping --> Pay[Redirect to Stripe Secure Payment]
    Pay --> Success{Payment Successful?}
    Success -->|No| PayError[Show Error & Return to Pay]
    PayError --> Pay
    Success -->|Yes| SaveOrder[Save Order & Update Stock in DB]
    SaveOrder --> End([End: Order Success View])
4. مخطط الحالة لدورة حياة الطلب (State Diagram)
يوضح حالات الطلب المختلفة داخل النظام:

stateDiagram-v2
    [*] --> Pending: Order Created (Awaiting Payment)
    Pending --> PaidProcessing: Payment Success (Stripe Webhook)
    Pending --> Cancelled: Payment Failed or Cancelled
    PaidProcessing --> Shipped: Dispatched to Delivery Agent
    Shipped --> Delivered: Delivered to Customer
    Cancelled --> [*]
    Delivered --> [*]
5. مخطط الفئات ونماذج البيانات (Class Diagram)
يوضح بنية النطاق البرمجي المتوافقة مع Entity Framework Core:

classDiagram
    class User {
        +String Id
        +String FullName
        +String Email
        +String Role
    }
    class Category {
        +int Id
        +String Name
    }
    class Product {
        +int Id
        +String Name
        +decimal Price
        +int StockQuantity
        +int CategoryId
    }
    class CartItem {
        +int Id
        +String UserId
        +int ProductId
        +int Quantity
    }
    class Order {
        +int Id
        +String UserId
        +DateTime OrderDate
        +decimal TotalAmount
        +String OrderStatus
        +String PaymentIntentId
    }
    class OrderItem {
        +int Id
        +int OrderId
        +int ProductId
        +int Quantity
        +decimal UnitPrice
    }

    Category "1" --> "*" Product : contains
    User "1" --> "*" CartItem : owns
    Product "1" --> "*" CartItem : added_to
    User "1" --> "*" Order : places
    Order "1" --> "*" OrderItem : includes
    Product "1" --> "*" OrderItem : ordered_in
