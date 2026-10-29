```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant UI as ASP.NET MVC View
    participant Cart as Cart Service
    participant Stripe as Stripe API
    participant DB as SQL Server

    Customer->>UI: Click Checkout
    UI->>Cart: Validate Cart Items
    Cart->>Stripe: Create Payment Intent
    Stripe-->>Cart: Webhook Token Success
    Cart->>DB: Save Order & Update Stock
    DB-->>UI: Transaction Confirmed
    UI-->>Customer: Order Success View

flowchart TD
    Customer[Customer] -->|Order Data| System[TechXpress System]
    Admin[Admin] -->|Manage Inventory| System
    Delivery[Delivery Agent] -->|Track Shipping| System
    System -->|Invoice| Customer
    System -->|Process Payment| Stripe[Stripe API]
    Stripe -->|Webhook| System

flowchart TD
    subgraph TechXpress [System Core]
        OP1[1. Auth & Identity]
        OP2[2. Catalog Search]
        OP3[3. Shopping Cart]
        OP4[4. Checkout & Order]
    end

    C[Customer] --> OP1
    C --> OP2
    C --> OP3
    C --> OP4
    OP4 --> DB[(SQL Server)]
    OP4 --> Stripe[Stripe API]

flowchart TD
    Start([Start]) --> Browse[Browse Products]
    Browse --> AddCart[Add to Cart]
    AddCart --> Check{Logged In?}
    Check -->|No| Login[Login via Identity]
    Login --> Ship[Enter Shipping]
    Check -->|Yes| Ship
    Ship --> Pay[Stripe Payment]
    Pay --> Success{Successful?}
    Success -->|No| Pay
    Success -->|Yes| Save[Save Order & Update DB]
    Save --> End([Order Success])

stateDiagram-v2
    [*] --> Pending: Order Created
    Pending --> PaidProcessing: Payment Success
    Pending --> Cancelled: Payment Failed
    PaidProcessing --> Shipped: Dispatched
    Shipped --> Delivered: Delivered
    Cancelled --> [*]
    Delivered --> [*]

classDiagram
    class User {
        +String Id
        +String FullName
        +String Email
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
    }
    class Order {
        +int Id
        +String UserId
        +DateTime OrderDate
        +decimal TotalAmount
        +String OrderStatus
    }

    Category "1" --> "*" Product : contains
    User "1" --> "*" Order : places


