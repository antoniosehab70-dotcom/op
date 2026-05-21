

الـ methodology بتاعة الـ web app pentesting هي قلب الشغل — مش مجرد قائمة خطوات، دي طريقة تفكير كاملة. هشرحلك كل phase بعمق وأوريك إزاي بتتعمل عملياً على ترجيت حقيقي.



دلوقتي هشرح كل phase بالتفصيل — إيه اللي بتعمله فيها، وإزاي بتعمله عملياً.

---

## 🔷 Phase 1 — Pre-Engagement (ما قبل التعاقد)

### إيه اللي بيحصل؟

دي أول حاجة وأهم حاجة قبل ما تمسّ أي ترجيت. مفيش سطر code، مفيش scan، مفيش أي حاجة — لحد ما تخلص المرحلة دي.

### عملياً بتعمل إيه؟

**أولاً: تحديد الـ Scope (النطاق)**

- بتسأل العميل: إيه التطبيقات المسموح باختبارها؟ إيه الـ URLs؟ فيه sub-domains تدخل فيها؟
- بتكتب الـ scope بالظبط عشان ما تعدّيش عليه

مثال عملي:

```
In-Scope:
  - app.example.com
  - api.example.com/v1/*
  - staging.example.com

Out-of-Scope:
  - admin.example.com (production admin panel)
  - Payment processor (third-party)
```

**ثانياً: الـ Authorization (التصريح)** لازم ياخد توقيع كتابي من صاحب التطبيق. من غير ورقة موقعة، أنت بتعمل جريمة — حتى لو التطبيق ضعيف. في الـ Capture The Flag (CTF) platforms زي TryHackMe ده بيتعمل أوتوماتيك لما تنضم للـ room.

**ثالثاً: الـ Rules of Engagement (RoE)** بتحدد:

- ساعات الاختبار (ممكن يكون في أوقات معينة بس عشان ما تأثرش على الـ production)
- هل مسموح تعمل DoS testing؟
- هل مسموح تعدّي من الـ web للـ server؟

**رابعاً: Risk Assessment** قبل ما تبدأ، بتفكر: لو الاختبار ده أثر على التطبيق، إيه التأثير؟ وبتاخد موافقة الإدارة.

---

## 🔷 Phase 2 — Information Gathering & Reconnaissance (جمع المعلومات)

### إيه اللي بتعمله؟

دي مرحلة الجمع الصامت — بتبص من بعيد قبل ما تقترب. كلما جمعت معلومات أكتر، كلما هجومك أذكى.

### عملياً على ترجيت حقيقي:

**Passive Recon (استطلاع سلبي) — من غير ما تلمس السيرفر:**

```bash
# WHOIS - مين صاحب الدومين
whois target.com

# DNS enumeration - إيه الـ subdomains
dnsenum target.com
subfinder -d target.com

# Google Dorks - معلومات على Google
site:target.com filetype:pdf
site:target.com inurl:admin
intitle:"index of" site:target.com
```

**Active Recon (استطلاع نشط) — بتكلم السيرفر:**

```bash
# Nmap - إيه البورتات المفتوحة والسيرفيسز
nmap -sV -sC -p 80,443,8080,8443 target.com

# Web fingerprinting - إيه التقنيات المستخدمة
whatweb target.com
curl -I target.com   # بيرجع الـ HTTP headers

# Technology detection
# أو Wappalyzer extension على المتصفح
```

**اللي بتدور عليه:**

- IP addresses والـ hosting provider
- Technologies: PHP? Node.js? WordPress? Apache? Nginx?
- Email addresses وأسماء موظفين (ممكن تستخدمها في social engineering)
- Hidden directories مش ظاهرة للزوار

---

## 🔷 Phase 3 — Threat Modeling (نمذجة التهديدات)

### إيه اللي بتعمله؟

بتسأل نفسك: "لو أنا هاكر، هدخل منين؟"

بتحلل:

- الـ **attack surface** (سطح الهجوم): كل نقطة دخول ممكنة — login forms، APIs، file uploads، search bars
- الـ **data flow** (تدفق البيانات): البيانات بتتروح فين؟ بتتخزن إزاي؟
- الـ **trust boundaries** (حدود الثقة): مين عنده صلاحيات إيه؟

### عملياً:

بترسم map ذهنية للتطبيق:

```
Entry Points:
├── /login (POST) → credentials
├── /api/search (GET) → query parameter
├── /upload (POST) → file upload
├── /profile/edit (PUT) → user data
└── /admin/* → restricted area

High Risk Areas:
├── Authentication
├── File Upload
└── API Endpoints
```

---

## 🔷 Phase 4 — Vulnerability Scanning (فحص الثغرات)

### إيه اللي بتعمله؟

بتستخدم أدوات آلية تفحص التطبيق وتطلعلك قائمة ثغرات محتملة.

### عملياً:

**Directory Discovery أول حاجة:**

```bash
# gobuster - بيدور على مجلدات ومسارات مخفية
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# dirsearch
dirsearch -u http://target.com -e php,html,js

# ffuf
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://target.com/FUZZ
```

**Vulnerability Scanning:**

```bash
# Nikto - بيشوف misconfigurations وثغرات معروفة
nikto -h http://target.com

# Burp Suite Active Scan (من الـ GUI)
# بترسل الـ request للـ scanner وبيجرب عليه
```

**النتيجة:** قائمة بـ "findings" محتملة زي:

- Directory listing enabled
- Outdated software version
- Missing security headers
- Potential SQL injection points

⚠️ **مهم جداً:** الـ scanner بيطلع **false positives** (نتائج غلط). مهمتك في الـ phase الجاية إنك تتحقق يدوياً.

---

## 🔷 Phase 5 — Manual Testing & Exploitation (الاختبار اليدوي)

### إيه اللي بتعمله؟

دي قلب الـ pentesting. بتاخد كل حاجة من الـ phases اللي فاتت وبتجرب استغلالها بشكل يدوي.

### عملياً:

**SQL Injection:**

```bash
# اختبار يدوي أول
# في الـ search bar مثلاً بتكتب:
' OR '1'='1
' OR '1'='1'--
' UNION SELECT null,null--

# لو أكدت الثغرة، تستخدم sqlmap
sqlmap -u "http://target.com/search?q=test" --dbs
```

**XSS:**

```javascript
// في أي input field:
<script>alert('XSS')</script>
<img src=x onerror=alert(1)>
"><script>alert(document.cookie)</script>
```

**Authentication Bypass:**

```
# Brute force login
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

**File Inclusion:**

```
# Path traversal
http://target.com/page?file=../../../etc/passwd
http://target.com/page?file=....//....//etc/passwd
```

---

## 🔷 Phase 6 — Authentication & Authorization Testing

### إيه اللي بتعمله؟

بتفحص:

- هل الـ password policy (سياسة الباسورد) قوية؟
- هل فيه **account lockout** (قفل الحساب بعد محاولات غلط)؟
- هل ممكن مستخدم عادي يوصل لصفحات الـ admin؟

### عملياً:

```bash
# Brute force test
# بتجرب 100 باسورد وبتشوف — هل الحساب بيتقفل؟

# IDOR (Insecure Direct Object Reference) - ثغرة Authorization
# مثلاً لو أنا User ID = 5
GET /api/profile/5    # هيرجع بياناتي
GET /api/profile/6    # المفروض يرفض — لو وصلت، فيه ثغرة IDOR
```

---

## 🔷 Phase 7 — Session Management Testing

### إيه اللي بتعمله؟

بتفحص الـ session tokens والـ cookies — هل هي آمنة؟

### عملياً في Burp Suite:

```
1. سجل دخولك وانظر للـ Cookie
2. افحص الـ Cookie في Burp:
   - هل فيها HttpOnly flag? (بيمنع JavaScript من قراءتها)
   - هل فيها Secure flag? (بيخليها تتبعت على HTTPS بس)
   - هل هي عشوائية كفاية؟ (مش متوقعة)
   
3. جرب Session Fixation:
   - احضر Session ID قبل Login
   - سجل دخولك
   - هل الـ Session ID تغير؟ (المفروض يتغير)
   
4. جرب Session Hijacking:
   - احضر Cookie من browser
   - استخدمها في browser تاني
   - هل اشتغلت؟
```

---

## 🔷 Phase 8, 9, 10 — المزيد من الاختبارات

**Information Disclosure:** بتبص في الـ error messages — لو السيرفر بيقولك "MySQL Error: Table 'users' doesn't exist"، ده بيقولك نوع الـ database والجداول الموجودة.

**Business Logic Testing:** مثال — لو بتشتري حاجة بـ 100 جنيه وغيّرت الـ quantity لـ -1، هتدفع -100 جنيه؟ بعض التطبيقات بتتصرف بشكل غريب.

**Client-Side Testing:** بتفحص الـ JavaScript والـ HTML في المتصفح — ممكن تلاقي API keys مخبية، أو DOM-based XSS.

---

## 🔷 Phase 11 — Reporting & Remediation

### إيه اللي بتكتبه في الـ Report؟

كل ثغرة لازم يكون فيها:

|العنصر|الوصف|
|---|---|
|**Vulnerability Name**|اسم الثغرة|
|**Severity**|خطورتها: Critical / High / Medium / Low|
|**Description**|وصف الثغرة وإزاي اكتشفتها|
|**Steps to Reproduce**|خطوات إعادة الثغرة|
|**Impact**|إيه اللي ممكن يعمله المهاجم|
|**Recommendation**|إزاي تُصلحها|
|**Evidence**|Screenshots أو requests|

**Risk Rating بيستخدم CVSS Score:**

- Critical: 9.0 - 10.0
- High: 7.0 - 8.9
- Medium: 4.0 - 6.9
- Low: 0.1 - 3.9

---

## 🔷 عملياً على ترجيت — الـ Workflow الكامل

لما هتاخد ترجيت جديد على AttackDefense أو TryHackMe، الـ workflow بيكون كده:

```
1. نقرأ الـ scope (إيه المسموح)
2. nmap -sV -sC target_ip           → بنشوف البورتات
3. whatweb / curl -I target         → بنشوف التقنيات
4. gobuster dir -u http://target    → بندور على مجلدات
5. nikto -h http://target           → بنفحص الثغرات الأساسية
6. Burp Suite → نبدأ يدوي          → نفحص الـ inputs
7. نجرب الثغرات اللي لقيناها
8. نوثق كل حاجة
```

---

## 📋 جدول المصطلحات

|English Term|المعنى بالعربي|النطق|
|---|---|---|
|Scope|النطاق|—|
|Rules of Engagement (RoE)|قواعد الاشتباك|آر-أوه-إي|
|Attack Surface|سطح الهجوم|—|
|Trust Boundary|حدود الثقة|—|
|False Positive|نتيجة إيجابية خاطئة|—|
|False Negative|نتيجة سلبية خاطئة|—|
|Directory Discovery|اكتشاف المجلدات|—|
|Wordlist|قائمة كلمات|—|
|Account Lockout|قفل الحساب|—|
|IDOR|مرجع مباشر غير آمن للكائنات|آي-دي-أوه-آر|
|CVSS Score|نظام تقييم نقاط الثغرات|سي-في-إس-إس|
|Severity|درجة الخطورة|سيفيريتي|
|Evidence|الدليل|—|
|Business Logic|منطق الأعمال|—|
|Client-Side|جانب العميل (المتصفح)|—|
|HttpOnly Flag|علامة HttpOnly للـ Cookie|—|
|Secure Flag|علامة Secure للـ Cookie|—|
|DOM-based XSS|XSS عبر الـ DOM|دي-أوه-إم|
|Path Traversal|اجتياز المسار|—|
|Brute Force|القوة العمياء|برووت فورس|

