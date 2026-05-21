تمام، اتسحبت المحضرة كلها. دلوقتي هشرحهالك بالطريقة الجديدة وهضيف أسئلة الامتحان في سياقها.

بس قبل ما أبدأ، هرد على سؤالك الأهم الأول:

---

## 📌 هتذاكر في شات واحد ولا شتات مختلفة؟

**الإجابة: كل محضرة في شات منفصل جوه المشروع.**

**ليه؟** عشان:

- كل محضرة بتبني على اللي قبلها، وربطهم ببعض هيبقى عن طريق إنك انت عارف السابق
- لو حطينا كل حاجة في شات واحد، الشات هيبقى ضخم جداً والمعلومات هتتخلط
- كل شات منفصل = وحدة تعليمية كاملة، تقدر ترجعلها لما تحتاج
- في نهاية كل محضرة هقولك "إيه اللي جاي في المحضرة الجاية" عشان الربط يكون في دماغك انت

**الربط بيحصل إزاي؟** انت بنفسك بتبني الصورة الكبيرة شات ورا شات — وأنا في كل شرح هذكّرك بالعلاقة بالمحاضرات السابقة.

---

# 🎯 شرح المحضرة: Intro to Advanced Web Application Penetration Testing

---

## 🗺️ الصورة الكبيرة الأول — إيه اللي هنتكلم فيه النهارده؟

تخيل إنك صُحفي بتحقق في جريمة. مش هتروح تعتقل حد من غير:

1. **تخطيط** (Pre-Engagement) — تعرف ليه بتحقق وإيه صلاحياتك
2. **استطلاع** (Reconnaissance) — تجمع معلومات عن المشتبه بيه
3. **خريطة** (Mapping & Crawling) — تعرف كل الأماكن اللي بيترددوا عليها
4. **تأمين الأدلة** (Session Security) — تحمي الأدلة من التلاعب

المحضرة دي بتحطلك الأساس النظري والعملي لـ Web App Pentesting.

---

## 📖 الجزء الأول: Web App Pentesting Methodology

### المفهوم العام

**Web Application Penetration Testing** 
(اختبار اختراق تطبيقات الويب) = محاكاة هجوم حقيقي على تطبيق ويب، بهدف اكتشاف الثغرات قبل ما المهاجم الحقيقي يلاقيها.

الكلمة المهمة هنا: **"Methodology"** — النطق: _مِث-أو-دولوجي_ — يعني منهجية أو طريقة عمل منظمة ومرتبة.

---

### ليه المنهجية مهمة؟ (مش بس كلام نظري)

| السبب                                                 | المعنى العملي                                |
| ----------------------------------------------------- | -------------------------------------------- |
| **Consistencyكون سيستن سي ** (ثبات واتساق)            | كل مرة بتعمل pentest بنفس الجودة، مش بالصدفة |
| **Comprehensive Coverage** (تغطية شاملة)              | مش هتفوتك ثغرة لأنك نسيت تتحقق منها          |
| **Efficiency** (كفاءة)                                | بتوفر وقت عشان عارف انت فين وبتعمل إيه       |
| **Risk Prioritization** (ترتيب الأولويات حسب الخطورة) | بتعرف إيه اللي تبدأ بيه أول                  |
| **Legal Compliance** (الامتثال القانوني)              | بتشتغل بصلاحية وفي إطار قانوني               |

---

### الـ 12 Phase (مرحلة) في منهجية Web App Pentesting

دي الصورة الكبيرة — كل مرحلة هتتعمق فيها في محضرات جاية:

```
1. Pre-Engagement          → الاتفاق والتخطيط قبل البدء
2. Information Gathering   → جمع معلومات عن الهدف
3. Threat Modeling         → نمذجة التهديدات المحتملة
4. Vulnerability Scanning  → مسح الثغرات بأدوات أوتوماتيك
5. Manual Testing          → اختبار يدوي واستغلال
6. Auth & Authorization    → اختبار المصادقة والصلاحيات
7. Session Management      → اختبار إدارة الجلسات
8. Information Disclosure  → كشف المعلومات الحساسة
9. Business Logic          → اختبار منطق التطبيق
10. Client-Side Testing    → اختبار كود JavaScript والـ DOM
11. Reporting              → تقرير النتائج والتوصيات
12. Post-Engagement        → اجتماع ما بعد الاختبار
```

> 🔗 **الربط بالخلفية بتاعتك:** انت شغلت على TryHackMe rooms وعملت Enumeration وExploitation — دي المراحل 2 و4 و5 بالظبط. دلوقتي بنتعلم الإطار الكامل اللي بتتحرك فيه.

---

### المنهجيات المعروفة في الصناعة

**PTES**
(نطق: _بي-تي-إي-إس_) — Penetration Testing Execution Standard = معيار تنفيذ اختبار الاختراق — بيغطي كل حاجة من البداية للنهاية

**OWASP WSTG**
(نطق: _أو-واسب ـ دبليو-إس-تي-جي_) — Web Security Testing Guide = دليل اختبار أمان الويب — المرجع الأشمل لـ web app pentesting تحديداً

**OWASP Top 10**
(نطق: _أو-واسب توب تن_) = قايمة بيتم تحديثها بانتظام فيها أخطر 10 ثغرات في تطبيقات الويب. ابتدت سنة 2003. دي زي الـ "قائمة الأكثر مطلوبين" في عالم الثغرات.

---

### 🎯 أسئلة الامتحان المرتبطة بالجزء ده

**سؤال 10:** What is the purpose of planning in a pentest?
✅ **الإجابة الصحيحة: Define scope and approach** **ليه؟**
لأن الـ Pre-Engagement والتخطيط = تحديد النطاق والمنهج، مش كسر باسووردات أو استغلال أنظمة.

**سؤال 11:** What does "scope" mean in a pentest?
✅ **الإجابة الصحيحة: Targets allowed to test** **Scope** (نطق: _سكوب_) = نطاق الاختبار = الأهداف المسموح اختبارها بالظبط.

---

## 📖 الجزء الثاني: Pre-Engagement Phase (مرحلة ما قبل الانخراط)

**النطق:** _بري-إنجيجمنت فيز_

### المفهوم

دي المرحلة اللي بتحصل **قبل** ما تلمس أي سيستم. هي الاتفاق القانوني والتقني بينك وبين العميل.

تخيلها زي العقد اللي بيوقعه الجراح مع المريض قبل العملية: "هعمل كذا وكذا، ومش مسؤول عن كذا".

### مكونات الـ Pre-Engagement

```
📄 Statement of Work (SoW)  → نطق: ستيتمنت أوف وورك
   = وثيقة العمل اللي بتحدد:
   ✔ Objectives & Scope    → الأهداف والنطاق
   ✔ Timeline & Milestones → الجدول الزمني
   ✔ Authorized Actions    → الأفعال المسموح بيها
   ✔ Deliverables          → المخرجات المطلوبة
```

### خطوات الـ Pre-Engagement بالتفصيل

**1. Understanding Project Objectives** 
(_أندرستاندينج بروجكت أوبجيكتيفز_) = فهم أهداف المشروع — عايزين يعرفوا إيه بالظبط؟

**2. Scope Definition**
(_سكوب ديفينيشن_) = تحديد النطاق — URLs إيه؟ تطبيقات إيه؟ إيه اللي خارج النطاق؟

**3. Authorization** 
(_أوثورايزيشن_) = التصريح والإذن القانوني — **بدونه انت مجرم وليس pentester!**

**4. Rules of Engagement 
(RoE)** (_رولز أوف إنجيجمنت_) = قواعد الاشتباك — امتى ممنوع تختبر؟ كيف تتواصل؟ وقت الاختبار إيه؟

**5. NDA** 
(_إن-دي-إيه_) — Non-Disclosure Agreement = اتفاقية عدم الإفصاح — بتضمن إنك مش هتشيل المعلومات بره

**6. Risk Assessment**
(_ريسك أسيسمنت_) = تقييم المخاطر — إيه اللي ممكن يحصل وتقبله العميل؟

**7. Engagement Kick-off** 
(_إنجيجمنت كيك-أوف_) = الانطلاق الرسمي — بدء الاختبار رسمياً

---

### 🎯 أسئلة الامتحان المرتبطة

**سؤال 12:** What is the pre-engagement phase? ✅ **الإجابة الصحيحة: Agreement before testing starts** = الاتفاق قبل بدء الاختبار. مش كتابة تقارير ومش استغلال ثغرات.

**سؤال 4 (اللي غلطت فيه):** A tester floods the application unintentionally during brute-force crawling. What phase failure is this? ✅ **الإجابة الصحيحة: Pre-engagement planning**

**ليه ده مهم جداً؟** لأن لو في مرحلة الـ Pre-Engagement اتفقنا صح، كان المفروض:

- نحدد حدود معدل الطلبات (Rate Limits)
- نتفق على وقت الاختبار
- نحدد ممنوع نضغط على البيئة الإنتاجية

الـ flooding مش فشل في مرحلة الاستغلال، ده فشل في التخطيط الأولي — لأن التخطيط الصح كان هيمنع الموقف ده أصلاً.

> ❗ **الدرس من السؤال ده:** أي حادثة بتحصل أثناء الاختبار وكان ممكن تتفادى بتخطيط أفضل = فشل في Pre-Engagement.

---

## 📖 الجزء الثالث: Reconnaissance & Web App Mapping

### Reconnaissance (نطق: _ريكو-نيسنس_)

= الاستطلاع — جمع معلومات عن الهدف قبل ما تلمسه

**إيه اللي بنبحث عنه؟**

- IP Addresses (عناوين IP)
- Email Addresses (عناوين إيميل)
- Hidden Directories (مجلدات مخفية)
- Web Technologies (التقنيات المستخدمة)
- Data Breaches (تسريبات بيانات سابقة)

### نوعان من الـ Reconnaissance:

|النوع|المعنى|مثال|
|---|---|---|
|**Passive Recon** (نطق: _باسيف ريكون_)|جمع معلومات من غير ما تتواصل مع السيرفر|Google Dorks، قراءة معلومات عامة|
|**Active Recon** (نطق: _أكتيف ريكون_)|التواصل المباشر مع الهدف|إرسال طلبات، nmap scan|

### أدوات مذكورة في المحضرة:

- **Google Dorks** (_جوجل دوركس_) = أوامر بحث متقدمة في Google لإيجاد معلومات حساسة
- **HTTrack** (_إتش-تي-تراك_) = أداة تنزيل مواقع كاملة للـ mapping
- **EyeWitness** (_آي-ويتنس_) = أداة تصوير Screenshots للمواقع تلقائياً

---

### Web App Mapping & Crawling

**Mapping** (_ماپينج_) = رسم خريطة التطبيق — كل الصفحات والـ Endpoints

**Crawling** (_كروولينج_ — نطق: _كرو-لينج_) = التجوال التلقائي في الموقع لاكتشاف كل المسارات

**نوعان:**

- **Passive Crawling** (سلبي) = بدون إرسال requests جديدة، بس مراقبة الـ traffic — زي Burp Suite Proxy
- **Active/Automated Crawling** (نشط) = إرسال requests تلقائية لاكتشاف الـ Endpoints — زي OWASP ZAP

---

### 🎯 أسئلة الامتحان المرتبطة

**سؤال 13:** What is web application mapping? ✅ **الإجابة الصحيحة: Finding all pages and endpoints** = اكتشاف كل الصفحات والـ Endpoints في التطبيق.

**سؤال 14:** What is crawling? ✅ **الإجابة الصحيحة: Automatically browsing the website** = التجوال التلقائي في الموقع.

**سؤال 15:** Which is an example of passive reconnaissance? ✅ **الإجابة الصحيحة: Reading public information** = لأن Passive Recon = مش بتبعت requests للسيرفر.

**سؤال 5:** During recon, you identify multiple environments (dev, staging, prod). What is the BEST approach? ✅ **الإجابة الصحيحة: Verify scope and test allowed environments** **Dev** = Development (تطوير)، **Staging** = بيئة تجريبية، **Prod** = Production (الإنتاج الحقيقي) دايماً ترجع للـ Scope Agreement عشان تتأكد إيه المسموح تختبره.

**سؤال 6:** You notice rate limiting during crawling. What is the BEST response? ✅ **الإجابة الصحيحة: Bypass using timing control or distributed requests** **Rate Limiting** (_ريت ليميتينج_) = تحديد عدد الطلبات المسموح بيها — مش معناها توقف، معناها تتعامل معاها بذكاء.

**سؤال 8:** Application behaves differently based on geolocation. What is the BEST action? ✅ **الإجابة الصحيحة: Test using VPNs or IP manipulation** **Geolocation** (_جيو-لوكيشن_) = الموقع الجغرافي. بعض التطبيقات بتتصرف مختلف حسب البلد — الحل: VPN أو تغيير الـ IP.

**سؤال 9:** A tester completes mapping but does not revisit it after new findings. What is the issue? ✅ **الإجابة الصحيحة: Missed attack surface due to non-iterative testing** **Attack Surface** (_أتاك سيرفيس_) = مساحة الهجوم = كل النقاط اللي ممكن تهاجمها. الـ Mapping مش بيحصل مرة واحدة — لازم يتعمل **Iteratively** (_إيتيريتيفلي_) = بشكل متكرر ومتجدد مع كل اكتشاف جديد.

---

## 📖 الجزء الرابع: Session Security (أمان الجلسات)

### المفهوم الأساسي

تخيل إنك دخلت بنك. الكاشير الأول طلب منك بطاقتك وهويتك وتحقق منك. بعدين انت ماشي في البنك، أي كاشير تاني مش بيطلب منك الهوية تاني — لأن في "جلسة نشطة" بينك وبين البنك.

تطبيقات الويب بتعمل نفس الحاجة بالـ **Session ID** و**Cookies**.

---

### Session IDs (نطق: _سيشن آي-ديز_)

= معرّفات الجلسة — رقم سري فريد بيعرّف جلستك على السيرفر

**مثال:** لما بتلوج إن على موقع، السيرفر بيديك Session ID زي "Session12345" — من هنا كل request انت بتبعته بيشيل الرقم ده، والسيرفر بيعرف إنك انت.

### Cookies (نطق: _كوكيز_)

= بيانات صغيرة السيرفر بيبعتها للبراوزر يحفظها، والبراوزر يبعتها تاني مع كل request

---

### 🔴 Session Hijacking (اختطاف الجلسة)

**النطق:** _سيشن هاي-جاكينج_

المهاجم بيسرق الـ Session Token الخاص بالضحية ويستخدمه عشان يتنكر فيه.

**طرق السرقة:**

- **Session Sniffing** (_سيشن سنيفينج_) = التقاط الـ Token وهو بيتبعت على شبكة غير مشفرة (HTTP مش HTTPS)
- **XSS** (Cross-Site Scripting) = حقن JavaScript خبيث يسرق الـ Token
- **Session Prediction** (_سيشن بريديكشن_) = تخمين الـ Token لو كان قصير أو متوقع

**تأثير الاختطاف:**

- سرقة بيانات
- الاستيلاء على الحساب (Account Takeover)
- عمليات مالية غير مصرح بيها

---

### 🔴 Session Fixation (تثبيت الجلسة)

**النطق:** _سيشن فيكسيشن_

المهاجم **هو** اللي بيحدد الـ Session ID — مش بيسرقه، بيختاره مسبقاً!

**الخطوات:**

1. المهاجم يحصل على Session Token من الموقع
2. يبعت للضحية رابط فيه الـ Token ده
3. الضحية تلوج إن باستخدام الرابط ده
4. دلوقتي المهاجم عارف الـ Session ID ويقدر يدخل بيه!

**الفرق الجوهري:**

- Hijacking = سرقة Session موجود
- Fixation = تحديد الـ Session مسبقاً قبل اللوجين

---

### 🎯 أسئلة الامتحان المرتبطة

**سؤال 1:** A session ID remains unchanged after login. What does this indicate? ✅ **الإجابة الصحيحة: Session fixation vulnerability** **ليه؟** لأن الـ Session ID المفروض يتغير بعد اللوجين (Session Regeneration). لو فضل نفسه = الموقع عرضة لـ Session Fixation.

**سؤال 2:** You capture a session cookie over HTTP and reuse it successfully. What vulnerability exists? ✅ **الإجابة الصحيحة: Session hijacking** كده انت عملت **Session Sniffing** وبعدين **Session Hijacking** — التقطت الـ Cookie على شبكة HTTP مش مشفرة.

**سؤال 7:** An application uses short session IDs (6 characters). What is the concern? ✅ **الإجابة الصحيحة: Predictability and brute-force feasibility** **Predictability** (_بريديكتابيليتي_) = قابلية التوقع. Session ID من 6 حروف = ممكن يتخمّن بسهولة.

**سؤال 3:** Application uses role-based access, but endpoints are accessible by changing parameters. Which issue is this? ✅ **الإجابة الصحيحة: IDOR** **IDOR** (_آي-دي-أو-آر_) = Insecure Direct Object Reference = وصول مباشر غير آمن للموارد. لو قدرت تغير parameter في الـ URL وتوصل لبيانات مش بتاعتك، ده IDOR.

---

## 🔑 جدول المصطلحات الشاملة

|المصطلح|النطق|المعنى|
|---|---|---|
|Penetration Testing|_بينيتريشن تستينج_|اختبار الاختراق|
|Methodology|_مِث-أو-دولوجي_|منهجية|
|Pre-Engagement|_بري-إنجيجمنت_|مرحلة ما قبل الانخراط|
|Scope|_سكوب_|نطاق الاختبار|
|Rules of Engagement (RoE)|_رولز أوف إنجيجمنت_|قواعد الاشتباك|
|Statement of Work (SoW)|_ستيتمنت أوف وورك_|وثيقة العمل|
|NDA|_إن-دي-إيه_|اتفاقية عدم الإفصاح|
|Reconnaissance|_ريكو-نيسنس_|الاستطلاع|
|Passive Recon|_باسيف ريكون_|استطلاع سلبي|
|Active Recon|_أكتيف ريكون_|استطلاع نشط|
|Crawling|_كرو-لينج_|التجوال التلقائي|
|Mapping|_ماپينج_|رسم خريطة التطبيق|
|Attack Surface|_أتاك سيرفيس_|مساحة الهجوم|
|Iterative Testing|_إيتيريتيف تستينج_|الاختبار المتكرر المتجدد|
|Session ID|_سيشن آي-دي_|معرّف الجلسة|
|Cookie|_كوكي_|ملف تعريف الارتباط|
|Session Hijacking|_سيشن هاي-جاكينج_|اختطاف الجلسة|
|Session Fixation|_سيشن فيكسيشن_|تثبيت الجلسة|
|Session Sniffing|_سيشن سنيفينج_|التقاط الجلسة|
|Session Prediction|_سيشن بريديكشن_|تخمين الجلسة|
|IDOR|_آي-دي-أو-آر_|وصول مباشر غير آمن|
|Rate Limiting|_ريت ليميتينج_|تحديد معدل الطلبات|
|Geolocation|_جيو-لوكيشن_|الموقع الجغرافي|
|OWASP|_أو-واسب_|منظمة أمان تطبيقات الويب|
|WSTG|_دبليو-إس-تي-جي_|دليل اختبار أمان الويب|
|PTES|_بي-تي-إي-إس_|معيار تنفيذ اختبار الاختراق|
|Endpoint|_إند-بوينت_|نقطة نهاية في التطبيق|
|Threat Modeling|_ثريت موديلينج_|نمذجة التهديدات|

---

## ⚡ الملخص السريع (Quick Summary)

```
المحضرة دي = الأساس الكامل لـ Web App Pentesting

1. المنهجية (Methodology)
   → 12 مرحلة من Pre-Engagement للـ Post-Engagement
   → المنهجيات: OWASP WSTG + PTES + OWASP Top 10

2. Pre-Engagement
   → التخطيط + الإذن القانوني + قواعد الاشتباك
   → أي حادثة ممكن كانت تتمنع بتخطيط أحسن = فشل Pre-Engagement

3. Reconnaissance & Mapping
   → Passive = مش بتلمس السيرفر
   → Active/Crawling = بتبعت requests
   → الـ Mapping لازم يكون Iterative

4. Session Security
   → Session Hijacking = سرقة Session موجود
   → Session Fixation = تحديد الـ Session مسبقاً
   → Session قصير = خطر Brute Force
```

---

## 🔭 إيه اللي جاي؟

المحضرة دي حطت الأساس. المحضرات الجاية هتغطي:

- **تفاصيل Authentication Attacks** (هجمات المصادقة)
- **XSS, SQL Injection, IDOR** بعمق
- **Burp Suite & ZAP** عملياً

كل حاجة شرحناها النهارده هتتربط بيها. 💪