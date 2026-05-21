
---

# 🔐 Authentication & Session Management Testing

### شرح الـ Slides من 1 لـ 31

---

## 📌 أولاً: فكرة الـ Course إيه؟

الـ Course ده بيتكلم عن إزاي تختبر أمان الـ **Authentication** (التحقق من الهوية) والـ **Session Management** (إدارة الجلسات) في الـ Web Applications.

الهدف منه إنك تعرف:

- إزاي الـ Authentication بيشتغل
- إزاي الـ Sessions بتشتغل
- وإزاي تلاقي الـ Vulnerabilities فيهم وتـ exploit إيه

---

## 📌 ثانياً: اللي المفروض تاخده من الـ Course ده

بعد ما تخلص المحتوى ده، المفروض تقدر:

١. **تشرح الـ Authentication والـ Session Management** وفرقهم
٢. **تلاقي الـ Flaws** في أنظمة الـ Authentication وتـ exploit إيها 
٣. **تلاقي وتـ exploit الـ Session Flaws** زي الـ Session Hijacking والـ Session Fixation والـ CSRF 
٤. **تفهم وتـ test الـ JWT والـ OAuth** وتلاقي مشاكلهم ٥. **تـ bypass الـ 2FA** وأنظمة الـ OTP

---

## 📌 ثالثاً: Authentication إيه بالظبط؟

<br>

### 🔑 تعريف الـ Authentication

الـ **Authentication** (المصادقة / التحقق من الهوية) هو الـ process اللي بيتأكد إن الـ User اللي بيحاول يدخل هو فعلاً مين بيقول إنه مين.

يعني ببساطة: لما بتكتب الـ Username والـ Password وبتضغط Login، الـ Server بيسألك "انت مين؟" وبـ verify إنك فعلاً الشخص ده.

<br>

### ⚖️ Authentication vs Authorization — الفرق المهم جداً

ده سؤال بيجي في كل امتحان وفي كل interview:

|الموضوع|Authentication|Authorization|
|---|---|---|
|بيسأل إيه؟|**انت مين؟**|**مسموحلك بإيه؟**|
|بيعمل إيه؟|بيـ verify الـ Identity (هويتك)|بيـ verify الـ Permissions (صلاحياتك)|
|الـ Process|بيتشيك على الـ Credentials (credentials = بيانات الدخول)|بيتشيك على الـ Roles والـ Permissions|
|مثال|تسجيل الدخول بالـ Username والـ Password|بعد الدخول، هل تقدر تشوف صفحة الـ Admin؟|

> 💡 **تخيلها كده:** الـ Authentication هي البواب اللي بيسألك "هويتك إيه؟"، والـ Authorization هو الـ System اللي بيقرر "هتدخل على أنهي أوضة؟"

<br>

---

## 📌 رابعاً: أنواع الـ Authentication Mechanisms

<br>

### 1️⃣ Password-Based Authentication

الأبسط والأشهر — بتديله **Username + Password**، وهو بيتحقق منك.

### 2️⃣ MFA — Multi-Factor Authentication (المصادقة متعددة العوامل)

بيجمع **عاملين أو أكتر** عشان يتأكد منك:

- **Something you know** (حاجة بتعرفها): الـ Password أو الـ PIN
- **Something you have** (حاجة معاك): موبايلك أو الـ Security Token
- **Something you are** (حاجة انت إيها): بصمة الإصبع أو وجهك — الـ Biometrics

### 3️⃣ 2FA — Two-Factor Authentication

نوع من الـ MFA بالظبط، بس بيستخدم **عاملين بس**. مثلاً: الـ Password + كود الـ SMS.

### 4️⃣ Token-Based Authentication

بدل ما تبعت الـ Password في كل request، الـ Server بيديك **Token** (رمز مشفر) بعد ما تـ login. وانت بتبعته في كل request بعد كده. أشهر نوعين: **JWT** (جيسون ويب توكن) و **OAuth Tokens**.

### 5️⃣ SSO — Single Sign-On (تسجيل الدخول الموحد)

بتـ login مرة واحدة وبتدخل على **تطبيقات كتير** من غير ما تـ login تاني. مثال: لما بتـ login بـ Google Account وبتدخل Gmail وYouTube وGoogle Drive كلهم من غير ما تكتب الـ Password تاني. البروتوكولات الشائعة: **SAML** و **OAuth**.

### 6️⃣ OTP — One-Time Password (كلمة المرور لمرة واحدة)

باسورد مؤقتة بتتبعت على موبايلك أو إيميلك وبتشتغل **مرة واحدة بس** ولفترة قصيرة.

---

## 📌 خامساً: Session Management — إيه وليه مهمة؟

<br>

### 🌐 ليه في الأصل في Session؟

الـ **HTTP** (البروتوكول اللي بتشتغل عليه كل الـ Web) هو **Stateless** — يعني كل request مستقلة عن اللي قبلها ولا بيتذكرش حاجة.

يعني لو انت دخلت على موقع وعملت Login، وبعدين طلبت صفحة تانية، الـ Server مش هيعرفك! هيتعامل معاك كأنك زائر جديد.

**الحل؟** الـ **Session Management** — ده الـ System اللي بيخلي الـ Server يتذكرك.

<br>

### ⚙️ الـ Session بتشتغل إزاي؟

```
[1] انت بتـ Login
        ↓
[2] الـ Server بيعمل Session ID مميزة ليك
        ↓
[3] الـ Session ID دي بتتبعت ليك (في Cookie أو URL أو HTTP Header)
        ↓
[4] في كل request جديدة، انت بتبعت الـ Session ID
        ↓
[5] الـ Server بيعرفك من الـ Session ID دي
        ↓
[6] لما بتـ Logout أو الـ Session تـ Expire، بتتمسح
```

<br>

### 🛡️ الـ Session Functions الأربعة

|الـ Function|الشرح|
|---|---|
|**Session Creation** (إنشاء الجلسة)|بعد الـ Login، الـ Server بيعمل Session ID مميزة|
|**Session Maintenance** (صيانة الجلسة)|بتـ remember دورك وصلاحياتك في كل الـ Requests|
|**Session Security** (أمان الجلسة)|حماية من الـ Hijacking والـ Fixation باستخدام HTTPS وـ Secure Cookies|
|**Session Termination** (إنهاء الجلسة)|بتتمسح لما تـ Logout أو بعد فترة خمول — الـ Timeout|

<br>

### 🔗 العلاقة بين الـ Authentication والـ Session Management

```
Authentication     →    Session Management
"انت مين؟"              "نتذكر إنك انت"
     ↓                        ↓
بتـ verify هويتك      بيـ maintain هويتك في كل request
```

الـ Cycle الكاملة بتكون كده:

1. **Authentication** → بتـ Login وبتـ verify هويتك
2. **Session Creation** → الـ Server بيعمل Session ليك
3. **Session Maintenance** → بتتعامل مع الموقع بدون ما تـ Login تاني
4. **Session Termination** → بتـ Logout أو الـ Session بتـ Expire
5. **Re-authentication** → لو رجعت، بتـ Login تاني

> 💡 **ملحوظة دقيقة:** مش كل الـ Sessions بتبدأ بعد الـ Login! بعض التطبيقات بتعمل Session حتى للزوار الـ Anonymous عشان تتذكر تفضيلاتهم زي اللغة المختارة.

---

## 📌 سادساً: Authentication Testing — إيه وإزاي؟

<br>

### 🎯 الـ Authentication Testing إيه؟

هو إنك كـ Pentester بتـ probe وبتـ exploit الـ Weaknesses في الـ Authentication System.

بتـ test:

- الـ **Login Forms** (نماذج الدخول)
- الـ **Password Reset** (إعادة تعيين الباسورد)
- الـ **Multi-Factor Authentication** (المصادقة متعددة العوامل)
- الـ **Account Lockouts** (قفل الحساب بعد محاولات خاطئة)

الهدف إنك تلاقي Flaws زي:

- Weak Password Policies (سياسات باسورد ضعيفة)
- Brute Force vulnerabilities (هجمات التخمين المتكررة)
- Credential Stuffing (استخدام قوائم باسوردات مسربة)
- Session Fixation
- Inadequate Token Handling (التعامل السيء مع الـ Tokens)

<br>

### 📚 الـ OWASP WSTG — المرجع الرسمي

الـ **OWASP WSTG** (اختصار Web Security Testing Guide) هو الدليل الرسمي اللي بيحدد كيفية الـ Testing بطريقة منظمة وموحدة.

ده جدول الـ Authentication Tests فيه لحد الـ Slide 31:

|الـ Test|الـ ID|الشرح|
|---|---|---|
|**Credentials over Encrypted Channel**|WSTG-ATHN-01|بتتأكد إن الـ Credentials بتتبعت على HTTPS مش HTTP عادي|
|**Default Credentials**|WSTG-ATHN-02|بتشوف لو في الـ Default Usernames/Passwords لسه شغالة (زي admin/admin)|
|**Weak Lock Out Mechanism**|WSTG-ATHN-03|بتـ test الـ Lockout — يعني بعد كام محاولة غلط بيتقفل الحساب؟|
|**Bypassing Authentication Schema**|WSTG-ATHN-04|بتلاقي ثغرات تخليك تـ bypass الـ Login خالص|
|**Vulnerable Remember Password**|WSTG-ATHN-05|بتـ test وظيفة "تذكرني" هل هي آمنة ولا بتكشف بيانات حساسة؟|
|**Browser Cache Weaknesses**|WSTG-ATHN-06|بتتأكد إن الـ Sensitive Data مش متخزنة في الـ Browser Cache|
|**Weak Password Policy**|WSTG-ATHN-07|بتـ examine الـ Password Policy — هل بيطلب كلمة مرور قوية ولا لأ؟|

---

## 📌 جدول المصطلحات

|English Term|المعنى بالعربي|النطق|
|---|---|---|
|Authentication|المصادقة / التحقق من الهوية|أوثينتيكيشن|
|Authorization|التفويض / تحديد الصلاحيات|أوثورايزيشن|
|Session Management|إدارة الجلسات|سيشن مانيجمنت|
|Session ID|معرّف الجلسة (رقم مميز للجلسة)|سيشن آي-دي|
|Session Hijacking|اختطاف الجلسة|سيشن هايجاكينج|
|Session Fixation|تثبيت الجلسة (هجوم بتثبت فيه الـ ID)|سيشن فيكسيشن|
|Session Timeout|انتهاء صلاحية الجلسة|سيشن تايم-أوت|
|Multi-Factor Authentication (MFA)|المصادقة متعددة العوامل|مالتي-فاكتور أوثينتيكيشن|
|Two-Factor Authentication (2FA)|المصادقة الثنائية|تو-فاكتور أوثينتيكيشن|
|Single Sign-On (SSO)|تسجيل الدخول الموحد|سينجل ساين-أون|
|One-Time Password (OTP)|كلمة مرور لمرة واحدة|أو-تي-بي|
|JSON Web Token (JWT)|رمز ويب مشفر بصيغة JSON|جيسون ويب توكن|
|OAuth|بروتوكول التفويض المفتوح|أو-أوث|
|SAML|بروتوكول تبادل بيانات المصادقة|سامل|
|Credential Stuffing|حشو بيانات الدخول (هجوم بقوائم مسربة)|كريدنشال ستافينج|
|Brute Force|هجوم القوة العمياء (تخمين متكرر)|بروت فورس|
|Biometrics|القياسات الحيوية (بصمة، وجه)|بايو-ميتريكس|
|WSTG|دليل اختبار أمان الويب من OWASP|دبليو-إس-تي-جي|
|Stateless|عديم الحالة (مش بيتذكر)|ستيت-ليس|
|Token|رمز مصادقة مشفر|توكن|
|Cookie|ملف تعريف الارتباط|كوكي|
|Lockout Mechanism|آلية قفل الحساب|لوك-أوت ميكانيزم|
|Browser Cache|الذاكرة المؤقتة للمتصفح|براوزر كاش|
|Default Credentials|بيانات الدخول الافتراضية|ديفولت كريدنشالز|

---

## 📌 ملخص النقاط العملية المهمة

1. **Authentication ≠ Authorization** — الأولى "انت مين؟" والتانية "مسموحلك بإيه؟" — الفرق ده أساسي
    
2. **HTTP Stateless** — عشان كده في Session Management، بدونها الـ Server مش هيعرفك في كل request
    
3. **Session ID** هو مفتاح الجلسة — لو اتسرق (Hijacking) أو اتعمل عليه Fixation، الـ Attacker هيقدر يدخل كأنه انت
    
4. **الـ OWASP WSTG** هو المرجع اللي بتشتغل بيه — حفظ الـ IDs (WSTG-ATHN-01 إلى 07) مهم للـ eWPTX
    
5. **الـ MFA والـ 2FA** مش معناهم إن الـ System آمن 100% — فيهم Bypass Techniques هتتعلمها في الـ Course
    
6. **الـ SSO** خطر لأنه لو الـ SSO Provider اتاخد، كل التطبيقات المرتبطة بيه اتاخدت معاه
    

