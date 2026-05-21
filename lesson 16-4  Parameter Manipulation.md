اتحفظ. نكمل دلوقتي بنفس الشرح بس بـ Obsidian format من الأول.

---

# Parameter Manipulation — Authentication Bypass

## الصورة الكبيرة

في الدرسين اللي فاتوا — Username Enumeration والـ Brute Force — كنا محتاجين نعرف الـ password.

الـ **Parameter Manipulation** (التلاعب بالمعاملات) — النطق: **باراميتر مانيبيوليشن** — فكرته مختلفة خالص.

بدل ما تجرب passwords — بتحاول **تخلي الـ application يصدق إنك authenticated أصلاً** عن طريق تغيير قيم في الـ request.

> الفكرة الجوهرية: لو الـ application بيثق في بيانات بيجت من الـ client من غير ما يتحقق منها server-side — ده ثغرة.

---

## الـ Parameter Manipulation في الـ Pentest Methodology

```
[Recon] → [Enumeration] → [Authentication Testing] → [Exploitation]
                                      ↑
                          Parameter Manipulation هنا
```

بيجي كجزء من **WSTG-ATHN-04** — اللي اتكلمنا عنه في المحاضرة — Bypassing Authentication Schema.

---

## فين بتلاقي الـ Parameters دي؟

### 1 — URL Parameters
```
http://target.com/dashboard?admin=false&userid=5
```
تغير `admin=false` لـ `admin=true` — لو الـ server بيصدق الـ URL بدون تحقق → دخلت.

### 2 — POST Body (جسم الطلب)
```
username=john&password=wrongpass&role=user
```
تغير `role=user` لـ `role=admin`.

### 3 — Cookies
```
session=abc123; isAdmin=false; userId=42
```
تغير `isAdmin=false` لـ `isAdmin=true`.

### 4 — HTTP Headers (ترويسات الطلب)
```
X-User-Role: user
X-Forwarded-For: 192.168.1.1
```
بعض الـ applications بتتحقق من الـ role في الـ header.

---

## التنفيذ العملي بـ Burp Suite

### الخطوة 1 — امسك الـ Request

افتح Burp → Intercept ON → اعمل login بأي بيانات → امسك الـ request.

### الخطوة 2 — ابعته لـ Repeater

كليك يمين → **Send to Repeater** — النطق: **ريبيتر**

الـ **Repeater** هو الـ module في Burp اللي بيخليك تعدل في الـ request وتبعته تاني وتشوف الـ response — من غير ما تعمل attack كامل.

مختلف عن الـ Intruder — الـ Repeater للاختبار اليدوي، الـ Intruder للـ automated.

### الخطوة 3 — عدّل وجرب

**مثال 1 — URL Parameter:**
```http
GET /dashboard?userid=5&admin=false HTTP/1.1
Host: target.com
```
غيّر لـ:
```http
GET /dashboard?userid=5&admin=true HTTP/1.1
```

**مثال 2 — POST Body:**
```http
POST /login HTTP/1.1

username=john&password=wrongpass&debug=false
```
غيّر لـ:
```http
username=john&password=wrongpass&debug=true
```
بعض الـ applications عندها **debug mode** (وضع التصحيح) — لما يكون `true` بيبايبس الـ authentication.

**مثال 3 — Cookie Tampering:**
```
Cookie: session=abc123; role=user
```
غيّر لـ:
```
Cookie: session=abc123; role=admin
```

### الخطوة 4 — حلّل الـ Response

بص على:
- هل الـ response اتغير؟
- هل ظهر content جديد؟
- هل الـ status code اتغير؟ — زي `403 Forbidden` بقى `200 OK`
- هل في redirect لـ admin panel؟

---

## أنواع الثغرات اللي بتلاقيها

### 1 — Hidden Parameters (باراميترات مخفية)
الـ developer حط parameter في الـ HTML بس عمله `type="hidden"` — يعني مش بيظهر في الصفحة بس موجود في الـ request.

```html
<input type="hidden" name="isAdmin" value="false">
```

في الـ Burp — بتشوفه وبتعدل فيه.

### 2 — Forced Browsing (التصفح القسري)
بتحاول توصل لـ URLs محمية مباشرة من غير ما تمر بالـ authentication.

```
http://target.com/admin/panel
http://target.com/admin/users
http://target.com/config/settings
```

لو الـ app مش بيتشيك إنت logged in ولا لأ في كل page — هتوصل.

### 3 — Mass Assignment (التعيين الجماعي)
في الـ frameworks الحديثة — الـ server أحياناً بياخد كل الـ parameters اللي في الـ request ويحطها في الـ database object تلقائياً.

يعني لو بعتت:
```
POST /register
username=john&password=pass&role=admin
```

وابليكيشن مش بيـ filter الـ parameters — ممكن يحفظ `role=admin` في الـ database!

---

## سيناريو كامل من البداية للنهاية

**الهدف:** `http://target.com`

**الخطوة 1 — Recon:**
بتلاقي login page على `/login`

**الخطوة 2 — امسك الـ request بـ Burp:**
```http
POST /login HTTP/1.1
Host: target.com

username=test&password=test
```

**الـ Response:**
```http
HTTP/1.1 302 Found
Location: /dashboard?user=test&admin=false
```

**الخطوة 3 — لاحظ الـ admin=false في الـ URL!**

**الخطوة 4 — ابعت GET request للـ Repeater وعدّل:**
```http
GET /dashboard?user=test&admin=true HTTP/1.1
```

**الـ Response:**
```http
HTTP/1.1 200 OK

Welcome to Admin Panel!
```

**دخلت Admin Panel من غير password صح!**

---

## الدفاع — عشان تفهم الثغرة من جوا

| الثغرة | السبب | الحل |
|--------|-------|------|
| Admin flag في الـ URL | الـ server بيثق في الـ client | التحقق Server-Side دايماً |
| Role في الـ Cookie | الـ Cookie مش signed | استخدام signed/encrypted cookies |
| Hidden parameters | الـ developer اعتمد على إخفاء UI | مش الإخفاء حماية — لازم validation |
| Forced Browsing | مفيش access control على كل page | تحقق من الـ session في كل request |
| Mass Assignment | الـ framework بياخد كل parameters | تحديد الـ allowed fields بالاسم |

---

## مصطلحات الدرس

| English Term | المعنى | النطق |
|-------------|--------|-------|
| Parameter Manipulation | التلاعب بمعاملات الطلب | باراميتر مانيبيوليشن |
| URL Parameter | متغير في رابط الـ URL | — |
| POST Body | جسم طلب الـ POST | — |
| HTTP Header | ترويسة الطلب | — |
| Repeater | أداة التعديل اليدوي في Burp | ريبيتر |
| Hidden Parameter | متغير مخفي في الـ HTML | — |
| Forced Browsing | التصفح القسري لصفحات محمية | فورسد براوزنج |
| Mass Assignment | التعيين الجماعي للـ parameters | ماس أسايمنت |
| Server-Side Validation | التحقق من البيانات على السيرفر | سيرفر-سايد فاليديشن |
| Access Control | التحكم في الوصول | أكسس كونترول |
| Debug Mode | وضع التصحيح — أحياناً بيبايبس الـ auth | ديباج مود |
| Signed Cookie | كوكي موقعة رقمياً تمنع التعديل | سايند كوكي |
| Status Code 403 | Forbidden — ممنوع الوصول | — |
| Redirect | إعادة التوجيه بعد الـ login | ريدايريكت |

---

## ملخص الفكرة العملية

> الـ Parameter Manipulation بيعتمد على مبدأ واحد — **الـ developer اعتمد على الـ client في تحديد الصلاحيات بدل السيرفر**. مهمتك كـ pentester إنك تدور على أي قيمة في الـ URL أو الـ POST body أو الـ Cookie أو الـ Headers — تعدل فيها وتشوف الـ application بيصدقها ولا لأ. الـ Burp Repeater هو أداتك الأساسية هنا.

---

**الخطوة الجاية:** Session Management Testing — Cookie Tampering وإزاي تحلل الـ Session tokens وتتلاعب فيها.

نكمل؟