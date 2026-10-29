# المرحلة الثالثة: تدفق البيانات والسلوك (Data Flow and Behavior) - منصة TechXpress

## 1. مخطط التسلسل لعملية الدفع وإتمام الطلب (Checkout & Stripe Payment Sequence Diagram)

يوضح هذا المخطط تفاعل العميل مع طبقات النظام المختلفة (MVC Views, Services, Stripe API, وSQL Server) عند إتمام عملية الشراء:

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
