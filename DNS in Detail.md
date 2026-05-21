
    

---

# 🔥 مشروع DNS in Detail – شرح كامل

## 1️⃣ الفكرة العامة

- **DNS (Domain Name System = نظام أسماء النطاقات)**  
    هو نظام بيحول أسماء المواقع (Domain Names) اللي بنكتبها زي `tryhackme.com` إلى **IP Addresses (عناوين رقمية)** زي `104.26.10.229`، علشان نقدر نتصل بالسيرفر من غير ما نحفظ أرقام معقدة.
    
- تخيلها كقصة: كل بيت له عنوان بريدي، وكل جهاز على الإنترنت له عنوان رقمي، وDNS هو دليل التليفونات اللي بيترجم الاسم للرقم.
    

---

## 2️⃣ رحلة طلب الموقع (DNS Lookup) – القصة كاملة

1. **Local Cache (الكاش المحلي)**
    
    - جهازك يسأل نفسه: “هل أعرف عنوان الموقع ده قبل كده؟”
        
    - لو موجود → يرجع لك مباشرة
        
    - لو مش موجود → يسأل الـ Resolver
        
2. **Resolver (محلل DNS)**
    
    - سيرفر وسيط غالبًا مزود الخدمة (ISP) أو Public DNS زي Google DNS (8.8.8.8)
        
    - وظيفة: يسأل عن الـ IP نيابة عنك
        
3. **Root Server (السيرفر الجذري)**
    
    - أعلى مستوى في الـ DNS
        
    - لو Resolver مش عارف → يسأل Root Server
        
    - Root Server بيقول: “روح اسأل TLD Servers”
        
4. **TLD Server (Top-Level Domain Server = سيرفر النطاق الأعلى)**
    
    - مسؤول عن الجزء الأخير من الدومين (.com, .org, .net)
        
    - وظيفة: يحدد Name Server المسؤول عن الدومين
        
5. **Authoritative Name Server (السيرفر المسؤول)**
    
    - عنده الإجابة النهائية → IP الفعلي للموقع
        
    - يرجع الإجابة للـ Resolver → ويرجع لجهازك → وتوصل المتصفح
        
6. **اتصال المتصفح بالموقع**
    
    - بعد ما الجهاز ياخد الـ IP → المتصفح يتصل بالسيرفر مباشرة
        

> كمخترق (Attacker): ممكن تستخدم كل مرحلة للاستطلاع (Reconnaissance)، اكتشاف Subdomains، أو البحث عن ثغرات.

---

## 3️⃣ Domain Hierarchy (هيكل الدومين)

- **Root (الجذر)** → أعلى مستوى
    
- **TLD (Top-Level Domain = النطاق الأعلى)** → آخر جزء على اليمين
    
- **Second-Level Domain (النطاق الثاني)** → اسم الموقع نفسه
    
- **Subdomain (النطاق الفرعي)** → أي شيء على الشمال
    

---

### 🔹 TLD (Top-Level Domain)

- أمثلة: `.com, .org, .edu, .gov`
    
- نوعين:
    
    1. **gTLD (Generic Top Level Domain = نطاق عام)** → للغرض العام، تجاري، تعليم، حكومي
        
    2. **ccTLD (Country Code Top Level Domain = نطاق الدولة)** → لمواقع بلد معينة، زي `.eg, .uk, .ca`
        
- دلوقتي فيه آلاف gTLD جديدة: `.online, .club, .biz`
    

---

### 🔹 Second-Level Domain

- اسم الموقع نفسه، مثال: `tryhackme` في `tryhackme.com`
    
- القواعد:
    
    - حروف a-z، أرقام 0-9، و`-`
        
    - الحد الأقصى: 63 حرف
        
    - لا يبدأ أو ينتهي بـ `-`، ولا يكون فيه `--` متتالية
        

---

### 🔹 Subdomain

- أي شيء على يسار Second-Level Domain
    
- مثال: `admin.tryhackme.com` → `admin` هو Subdomain
    
- ممكن يكون متعدد: `jupiter.servers.tryhackme.com`
    
- القواعد:
    
    - نفس Second-Level
        
    - طول كل Subdomain ≤ 63 حرف
        
    - طول الدومين كامل ≤ 253 حرف
        

> كمخترق: غالبًا Subdomains فيها ثغرات أكثر من الموقع الرئيسي، وده أهم هدف في Reconnaissance.

---

## 4️⃣ طول وعلامات Subdomain

- أقصى طول Subdomain: **63 حرف**
    
- الرموز المسموح بها: a-z, 0-9, `-`
    
- الرموز الممنوعة: `_` أو أي رمز خاص غير `-`
    

---

## 5️⃣ أدوات عملية

- **nslookup** → لمعرفة IP من الدومين
    
- **dig** → لمعرفة كل Records (سجلات) DNS
    
- **host** → مشابهة لـ dig و nslookup
    
- **Subdomain enumeration tools (اكتشاف نطاقات فرعية)**:
    
    - `sublist3r`
        
    - `amass`
        
    - `assetfinder`
        

---

## 6️⃣ مفاهيم مهمة

- **Cache (الذاكرة المؤقتة)** → يحفظ الدومينات للإجابة السريعة
    
- **Resolver (محلل DNS)** → يسأل عن الدومين نيابة عنك
    
- **Root Server (الجذر)** → أعلى مستوى
    
- **TLD (النطاق الأعلى)** → آخر جزء في الدومين
    
- **Second-Level Domain (النطاق الثاني)** → اسم الموقع نفسه
    
- **Subdomain (النطاق الفرعي)** → أي جزء إضافي على الشمال
    
- **AXFR / Zone Transfer (نقل المنطقة)** → طريقة لأخذ كل الدومينات الفرعية من السيرفر المسؤول
    

---

## 7️⃣ أمثلة عملية

- `dig tryhackme.com` → يسأل DNS مباشرة
    
- `dig @ns1.tryhackme.com tryhackme.com` → يسأل Name Server المسؤول
    
- Zone Transfer: لو السيرفر سمح بـ AXFR → تاخد كل Subdomains مرة واحدة
    

---

## 8️⃣ الخلاصة

- DNS = دليل التليفونات للإنترنت
    
- رحلة الطلب: جهازك → Resolver → Root → TLD → Name Server → IP → المتصفح
    
- Domain Hierarchy = Root → TLD → Second-Level → Subdomain
    
- Subdomains = كنز المخترقين (هدف Recon)
    
- طول Subdomain ≤ 63 حرف، طول الدومين كله ≤ 253 حرف
    
- أدواتك المهمة: nslookup, dig, host, sublist3r, amass, assetfinder
    

---

لو تحب، أقدر أعمللك **رسم توضيحي كامل للرحلة + Domain Hierarchy + Subdomains** بحيث كل حاجة واضحة جدًا في صفحة واحدة 🔥

تحب أعملهولك دلوقتي؟