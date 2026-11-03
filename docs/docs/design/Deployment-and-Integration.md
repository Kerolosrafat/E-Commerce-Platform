مخطط النشر (Deployment Diagram)
```mermaid
graph TD
    subgraph Client Environment
        Browser[Web Browser / Mobile Viewport]
    end

    subgraph Cloud Hosting Environment [Azure App Service / Somee IIS]
        WebServer[ASP.NET Core MVC Web Application Server]
    end

    subgraph Database Server
        SQLServer[(Microsoft SQL Server 2022)]
    end

    subgraph External Third-Party Services
        Stripe[Stripe Payment Gateway & Webhooks]
    end

    Browser -->|HTTPS / TLS 1.3 Requests| WebServer
    WebServer -->|Entity Framework Core / LINQ| SQLServer
    WebServer -->|HTTPS / API Calls & Webhooks| Stripe
```
مخطط المكونات (Component Diagram)
```mermaid
graph TD
    subgraph Presentation Layer
        Views[ASP.NET Core MVC Views & Razor Pages]
        Controllers[MVC Controllers & ViewModels]
    end

    subgraph Business Logic Layer - BLL
        AuthService[ASP.NET Identity & RBAC Service]
        CartService[Shopping Cart & Session Management]
        PaymentService[Stripe Payment & Webhook Processor]
    end

    subgraph Data Access Layer - DAL
        UoW[Unit of Work & Generic Repository]
        EF[Entity Framework Core ORM]
    end

    subgraph Database Layer
        DB[(SQL Server Database)]
    end

    Views --> Controllers
    Controllers --> AuthService
    Controllers --> CartService
    Controllers --> PaymentService
    
    CartService --> UoW
    PaymentService --> UoW
    UoW --> EF
    EF --> DB



```
4. التقنيات النهائية المستخدمة (Tech Stack)
5. <img width="950" height="689" alt="image" src="https://github.com/user-attachments/assets/7ee512b2-c91e-46ae-a5ca-71e7746d9147" />
5. توثيق المسارات البرمجية وواجهات التحكم (Controller Endpoints)
6.
<img width="1031" height="734" alt="image" src="https://github.com/user-attachments/assets/f54ba5a5-0ba2-4a5b-922b-51b1f12d65eb" />

6. خطة الاختبار (Testing Plan)
لضمان جودة النظام وخلوه من الأخطاء قبل التسليم النهائي، تم وضع خطة اختبار شاملة تتضمن:

Unit Testing (الاختبارات الوحدة):

اختبار دوال الحسابات الرياضية في سلة التسوق (إجماليات الأسعار، الضرائب، وخصومات الكمية).

التحقق من سلامة عمل الـ Repositories واستعلامات الـ LINQ.

Integration Testing (اختبار التكامل):

اختبار ربط الـ Entity Framework بقاعدة بيانات SQL Server والتأكد من نجاح الـ Migrations واستقرار العلاقات بين الجداول.

اختبار الربط الآمن مع Stripe Test API والتأكد من تطابق الـ Webhooks.

Payment & Webhook Simulation:

محاكاة عمليات الدفع الناجحة والفاشلة باستخدام بطاقات اختبار Stripe، والتحقق من التحديث الفوري لحالة الطلب في قاعدة البيانات.

User Acceptance Testing (UAT):

اختبار صلاحيات الوصول المختلفة (RBAC) والتأكد من أن العميل، الأدمن، وعامل التوصيل يمتلكون الصلاحيات المخصصة لهم فقط دون تداخل.

7. استراتيجية النشر (Deployment Strategy)
إعدادات بيئة العمل (Environment Variables):

تخزين بيانات الاتصال الحساسة بقاعدة البيانات (ConnectionStrings) ومفاتيح السر الخاصة ببوابة الدفع (StripeSettings:SecretKey) بشكل آمن داخل ملفات الإعدادات السحابية (appsettings.json أو لوحة تحكم سحابة Azure).

ترحيل قاعدة البيانات (Database Migration Strategy):

تشغيل أوامر النشر المحدثة لتطبيق الجداول على سيرفر الإنتاج:


dotnet ef database update --configuration Release
إدارة النشر السحابي (Cloud Hosting):

نشر التطبيق على بيئة Azure App Service أو خادم Somee IIS، مع تفعيل شهادات الحماية HTTPS / TLS 1.3 لتأمين بيانات العملاء والبطاقات الائتمانية أثناء الدفع.



