# المرحلة الثالثة: تدفق البيانات والسلوك (Data Flow and Behavior) - منصة TechXpress

يوضح هذا المستند التصميم الهيكلي وتدفق العمليات البرمجية والسلوكية لنظام التجارة الإلكترونية **TechXpress** بناءً على معمارية N-Tier و ASP.NET Core MVC.

---

## 1. مخطط التسلسل لعملية الدفع وإتمام الطلب (Checkout & Stripe Payment Sequence Diagram)

يوضح هذا المخطط تفاعل العميل مع طبقات النظام المختلفة عند إتمام عملية الشراء:

```mermaid
sequenceDiagram
    autonumber
    actor Customer as العميل (Customer)
    participant UI as واجهة المتجر (ASP.NET MVC View)
    participant Cart as خدمة السلة (Cart Service)
    participant Stripe as بوابة الدفع (Stripe API)
    participant DB as قاعدة البيانات (SQL Server)

    Customer->>UI: الضغط على زر "إتمام الطلب والدفع"
    UI->>Cart: التحقق من توفر عناصر سلة التسوق
    Cart->>Stripe: إنشاء طلب دفع آمن (Payment Intent)
    Stripe-->>Cart: إرسال توكن نجاح المعاملة (Webhook Token)
    Cart->>DB: حفظ بيانات الطلب وتحديث المخزون (Unit of Work)
    DB-->>UI: تأكيد إتمام المعاملة بنجاح
    UI-->>Customer: توجيه العميل إلى صفحة نجاح الطلب (Order Success)



flowchart TD
    Customer[العميل - Customer] -->|طلب المنتجات وبيانات الدفع| System[نظام TechXpress E-commerce]
    Admin[المسؤول - Admin] -->|إدارة المنتجات والمخزون| System
    Delivery[مسؤول التوصيل - Delivery] -->|متابعة حالة الشحن| System
    System -->|تأكيد الطلب والفاتورة| Customer
    System -->|طلب معالجة الدفع| Stripe[بوابة الدفع Stripe API]
    Stripe -->|تأكيد المعاملة Webhook| System



flowchart TD
    subgraph TechXpress System
        OP1[1. إدارة المستخدمين والهوية]
        OP2[2. تصفح والبحث في الكتالوج]
        OP3[3. إدارة سلة التسوق]
        OP4[4. إتمام الطلب والدفع]
    end

    C[العميل] -->|تسجيل وصلاحيات| OP1
    C -->|بحث وتصفية| OP2
    C -->|إضافة وتعديل السلة| OP3
    C -->|إرسال الطلب والدفع| OP4
    OP4 --> DB[(SQL Server DB)]
    OP4 --> Stripe[Stripe API]






flowchart TD
    Start([البداية]) --> Browse[تصفح المنتجات والبحث]
    Browse --> AddCart[إضافة المنتجات إلى السلة]
    AddCart --> Checkout{هل المستخدم مسجل دخول؟}
    Checkout -->|لا| Login[تسجيل الدخول / إنشاء حساب عبر ASP.NET Identity]
    Login --> Shipping[إدخال عناوين وشحن الطلب]
    Checkout -->|نعم| Shipping
    Shipping --> Pay[الانتقال لصفحة الدفع الآمن Stripe]
    Pay --> Success{هل نجح الدفع؟}
    Success -->|لا| PayError[عرض رسالة خطأ والعودة للدفع]
    PayError --> Pay
    Success -->|نعم| SaveOrder[حفظ الطلب وتحديث المخزون في SQL Server]
    SaveOrder --> End([النهاية: عرض صفحة نجاح الطلب])








stateDiagram-v2
    [*] --> Pending: إنشاء الطلب بانتظار الدفع
    Pending --> PaidProcessing: نجاح الدفع عبر Stripe (Webhook)
    Pending --> Cancelled: فشل الدفع أو إلغاء الطلب
    PaidProcessing --> Shipped: تسليم الطلب لموظف التوصيل
    Shipped --> Delivered: وصول الطلب للعميل بنجاح
    Cancelled --> [*]
    Delivered --> [*]



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

    Category "1" --> "*" Product : يحتوي على
    User "1" --> "*" CartItem : يمتلك
    Product "1" --> "*" CartItem : يضاف إلى
    User "1" --> "*" Order : ينشئ
    Order "1" --> "*" OrderItem : يتضمن
    Product "1" --> "*" OrderItem : يطلب في



