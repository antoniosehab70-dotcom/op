# JWT من الصفر — الشرح الكامل

---

## أولاً — إيه الـ JWT؟

### المشكلة اللي JWT جاء يحلها

HTTP
بطبيعته **Stateless** — يعني كل request مستقل، السيرفر مش بيتذكر مين انت.

يعني كل ما بتبعت request، السيرفر بيسأل:

> "مين انت؟"

الحل القديم كان **Sessions** — السيرفر يحفظ عنده إنك logged in.

الحل الحديث هو **JWT** — انت بتحمل إثبات هويتك معاك في كل request.

---

### الـ JWT بيتكون من إيه؟

```
HEADER.PAYLOAD.SIGNATURE
```

**مثال حقيقي:**

```
eyJhbGciOiJSUzI1NiJ9
.
eyJzdWIiOiJ3aWVuZXIifQ
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

كل جزء Base64 — تقدر تفكه في ثانية.

---

### الـ Header

```json
{
  "alg": "RS256",
  "kid": "my-key-id"
}
```

بيقول:

- `alg` — الخوارزمية المستخدمة في الـ Signature
- `kid` — ID بتاع المفتاح

---

### الـ Payload

```json
{
  "sub": "wiener",
  "role": "user",
  "exp": 1778922420
}
```

بيقول:

- `sub` — مين الـ user
- `role` — صلاحياته
- `exp` — امتى الـ token بينتهي

> **مهم:** الـ Payload مش متشفر — أي حد يقدر يقرأه. بس مينفعش يغيّره من غير ما يكسر الـ Signature.

---

### الـ Signature

ده الجزء اللي بيثبت إن الـ token أصلي ومش اتغيّر.

---

## ثانياً — أنواع الخوارزميات

### Symmetric — مفتاح واحد

```
نفس المفتاح بيوقّع وبيتحقق
```

مثال: **HS256**

```
السيرفر عنده secret = "mysecret"
التوقيع:    HMAC(Header + Payload, "mysecret") = Signature
التحقق:     HMAC(Header + Payload, "mysecret") = نفس الـ Signature؟
```

**المشكلة:** لو حد عرف الـ secret — يقدر يعمل أي token عايزه.

---

### Asymmetric — مفتاحين

```
Private Key  →  للتوقيع فقط (سري، عند السيرفر بس)
Public Key   →  للتحقق فقط (متاح لأي حد)
```

مثال: **RS256**

```
التوقيع:    RSA_Sign(Header + Payload, Private_Key) = Signature
التحقق:     RSA_Verify(Header + Payload, Signature, Public_Key) = صح/غلط
```

**الميزة:** حتى لو عرفت الـ Public Key — مش تقدر توقّع من غير الـ Private Key.

---

### الفرق ببساطة

||Symmetric (HS256)|Asymmetric (RS256)|
|---|---|---|
|المفاتيح|واحد|اتنين|
|التوقيع|بالـ secret|بالـ Private Key|
|التحقق|بنفس الـ secret|بالـ Public Key|
|الـ secret ظاهر؟|لأ|Public Key ظاهر — Private سري|

---

## ثالثاً — السيناريو الكامل من اللحظة الأولى

### لما السيرفر بيتجهّز

السيرفر بيعمل **Key Pair** مرة واحدة:

```
Private Key  →  محفوظ على السيرفر سري
Public Key   →  متاح على /jwks.json لأي حد
```

---

### لما بتعمل Login

```
1. بتبعت: POST /login  username=wiener  password=peter

2. السيرفر بيتحقق من الـ credentials في الـ Database

3. لو صح — السيرفر بيبني الـ JWT:

   Header  = { "alg": "RS256", "kid": "abc123" }
   Payload = { "sub": "wiener", "exp": 1778922420 }

4. السيرفر بيوقّع:
   Signature = RSA_Sign(Header + Payload, Private_Key)

5. بيبعتلك الـ JWT كامل:
   eyJhbGc....eyJzdWI....SflKxw....

6. انت بتحتفظ بيه في الـ Cookie
```

---

### لما بتبعت أي Request تاني

```
1. بتبعت: GET /my-account
   Cookie: session=eyJhbGc....

2. السيرفر بيقسّم الـ JWT لـ 3 أجزاء

3. بيقرأ الـ Header:
   - alg = RS256  → هستخدم RSA للتحقق
   - kid = abc123 → هجيب المفتاح ده

4. بيجيب الـ Public Key المقابل لـ kid = abc123

5. بيتحقق:
   RSA_Verify(Header + Payload, Signature, Public_Key)
   لو صح = يكمل
   لو غلط = 401

6. بيقرأ الـ Payload:
   sub = wiener → ده الـ user
   exp = لسه ما انتهاش → تمام

7. بيرد عليك بصفحة حسابك
```

---

## رابعاً — التحقق في الحالتين

### في HS256 (Symmetric)

```
التوقيع:
HMAC(Header + Payload, secret) = Signature

التحقق:
HMAC(Header + Payload, secret) = X
X == Signature؟  →  صح ✅ / غلط ❌
```

السيرفر بيعيد الحساب بنفس الـ secret ويقارن.

---

### في RS256 (Asymmetric)

```
التوقيع:
RSA_Sign(Header + Payload, Private_Key) = Signature

التحقق:
RSA_Verify(Header + Payload, Signature, Public_Key) = صح/غلط
```

مش بيعيد الحساب — بيستخدم خاصية رياضية في RSA إن الـ Public Key يقدر يتحقق من توقيع الـ Private Key.

---

## خامساً — السيرفر بيبص على إيه في التحقق؟

### الخطوات بالترتيب

```
1. الـ Format صح؟
   لازم يكون HEADER.PAYLOAD.SIGNATURE
   لو مش كده → رفض فوري

2. الـ Header
   - alg    → أي خوارزمية أستخدم؟
   - kid    → أي مفتاح أجيب؟
   - jku    → أجيب المفتاح من الرابط ده؟
   - jwk    → المفتاح موجود هنا جوه الـ Header؟

3. بيجيب المفتاح المناسب

4. بيتحقق من الـ Signature
   لو غلط → 401 فوري

5. الـ Payload
   - exp  → الـ token انتهى؟
   - iss  → مين اللي أصدره؟
   - sub  → مين الـ user؟

6. لو كل حاجة تمام → يكمل الـ request
```

---

## ليه الـ Algorithm Confusion خطيرة؟

الكود الغلط:

```python
# بياخد الـ algorithm من الـ token — خطأ قاتل
alg = token.header["alg"]
key = get_key(token.header["kid"])
verify(token, key, alg)
```

انت بتغيّر `alg` من `RS256` لـ `HS256` — الكود بيعمل:

```python
HMAC(Header + Payload, public_key) = X
X == Signature؟
```

السيرفر بقى بيتحقق بـ HMAC باستخدام الـ **Public Key كـ secret.**

انت عارف الـ Public Key من `/jwks.json` — فتعمل نفس الـ HMAC — والـ Signature تتطابق.

الكود الصح:

```python
# بيحدد الـ algorithm هو — صح
verify(token, public_key, algorithm="RS256")
# مهما غيّرت في الـ Header — هيستخدم RS256 دايماً
```

---

## الصورة الكاملة

```
RS256 عادي:
انت: Header(alg=RS256) + Payload + Signature(بالـ Private Key)
السيرفر: تحقق بالـ Public Key ← صح لأن Private وقّع

Algorithm Confusion:
انت: Header(alg=HS256) + Payload + Signature(بالـ Public Key كـ HMAC secret)
السيرفر الغلط: شايف HS256 → بيتحقق بـ HMAC(Public Key) ← صح!
لأنك انت وقّعت بنفس الـ Public Key
```

---

دلوقتي الصورة واضحة تماماً؟ 🎯