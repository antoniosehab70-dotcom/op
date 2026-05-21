تمام! هشرح الفكرة بعمق وبعدين الخطوات كاملة. 🎯

---

# 🧠 الفكرة بعمق — Password Brute-force via Password Change

---

## أولاً — إيه المشكلة الأساسية؟

الموقع عنده صفحة **Change Password** فيها 3 Fields:

```
1) Current Password   ← كلمة السر الحالية
2) New Password 1     ← كلمة السر الجديدة
3) New Password 2     ← تأكيد كلمة السر الجديدة
```

السيرفر بيتحقق من الـ Fields دي بترتيب معين، وبيرد برسائل مختلفة في كل حالة. الرسائل المختلفة دي هي **المشكلة** — لأنها بتكشف معلومات.

---

## ثانياً — إيه الرسائل المختلفة وإيه معناها؟

```
┌──────────────────────┬──────────────────────┬─────────────────────────────┐
│   Current Password   │   New Pass 1 & 2     │        الرسالة              │
├──────────────────────┼──────────────────────┼─────────────────────────────┤
│      غلط             │    متطابقين          │   Account locked ❌         │
├──────────────────────┼──────────────────────┼─────────────────────────────┤
│      غلط             │    مختلفين           │  Current password incorrect │
├──────────────────────┼──────────────────────┼─────────────────────────────┤
│      صح              │    مختلفين           │  New passwords do not match │
└──────────────────────┴──────────────────────┴─────────────────────────────┘
```

### الرسالة المهمة جداً:

```
"New passwords do not match"
     ↑
دي بتظهر بس لما الـ Current Password صح!
```

يعني لو بعتنا آلاف الـ Passwords وشيلنا على الرسالة دي، أول ما تظهر = ده الباسورد الصح!

---

## ثالثاً — ليه الثغرة دي موجودة؟

### السبب 1 — Information Leakage في الرسائل

```
المفروض السيرفر يقول:
"حصل خطأ، حاول تاني"
← مش بيكشف أي معلومة

الفعلي:
"Current password incorrect"  ← باسورد غلط
"New passwords do not match"  ← باسورد صح! 😱
```

### السبب 2 — مفيش Rate Limiting على Change Password

```
السيرفر سامحلك تجرب:
password1 ❌
password2 ❌
password3 ❌
...
password1000 ✅
من غير ما يبلوكك!
```

### السبب 3 — الـ Username في الـ Request قابل للتغيير

```
الـ Request فيه:
username=wiener  ← تقدر تغيره لـ carlos!

يعني تقدر تحاول تغيير باسورد
أي يوزر تاني من غير ما تكون مسجل دخول بيه
```

---

## رابعاً — إيه الفرق بين الـ Lab ده وباقي الـ Labs؟

```
┌─────────────────────────────────────────────────────┐
│           مقارنة طرق الـ Brute Force                │
├──────────────────┬──────────────────────────────────┤
│   Login Page     │   Change Password Page           │
├──────────────────┼──────────────────────────────────┤
│ بيجرب Login      │ بيجرب Current Password           │
│ مباشرة           │ في صفحة التغيير                  │
├──────────────────┼──────────────────────────────────┤
│ عادةً فيه        │ ممكن مفيش                        │
│ Rate Limit       │ Rate Limit                       │
├──────────────────┼──────────────────────────────────┤
│ Account Lockout  │ بيتجنب الـ Lockout               │
│ ممكن يحصل       │ لأننا بنخلي الـ New Passwords     │
│                  │ مختلفين عشان نتجنب القفل         │
└──────────────────┴──────────────────────────────────┘
```

---

## خامساً — إزاي بنتجنب الـ Account Lockout؟

ده الجزء الذكي في الـ Lab:

```
لو بعتنا:
current-password=WRONG&new-password-1=abc&new-password-2=abc
     ↑ غلط              ↑ متطابقين
→ Account Locked! ❌

بس لو بعتنا:
current-password=WRONG&new-password-1=abc&new-password-2=xyz
     ↑ غلط              ↑ مختلفين
→ "Current password incorrect" فقط ✅
→ مفيش Lock!
```

يعني بمجرد إننا نخلي الـ New Passwords مختلفين، السيرفر مش بيقفل الأكونت حتى لو الباسورد غلط!

---

# 🛠️ الخطوات العملية بالتفصيل

---

## الخطوة 1 — افهم الـ Change Password Request

**افعل:**

1. افتح الـ Lab
2. عمل Login بـ `wiener:peter`
3. روح **My Account**
4. في صفحة Change Password ادخل:
    - Current Password: `peter`
    - New Password 1: `abc`
    - New Password 2: `xyz`
5. اضغط **Change Password**

**في Burp روح:** `Proxy → HTTP History`

دور على `POST /my-account/change-password`

هتلاقي الـ Request شكله كده:

```
username=wiener&current-password=peter&new-password-1=abc&new-password-2=xyz
```

**ليه عملنا كده؟** عشان نشوف الـ Request بالكامل ونفهم الـ Parameters قبل ما نبعته لـ Intruder.

---

## الخطوة 2 — تأكد من الرسائل

**قبل ما تبعت لـ Intruder، تأكد من الرسائل دي:**

جرب 3 سيناريوهات:

```
السيناريو 1:
current-password=WRONG&new-password-1=abc&new-password-2=abc
→ المتوقع: Account locked

السيناريو 2:
current-password=WRONG&new-password-1=abc&new-password-2=xyz
→ المتوقع: "Current password is incorrect"

السيناريو 3:
current-password=peter&new-password-1=abc&new-password-2=xyz
→ المتوقع: "New passwords do not match"
```

**ليه مهم نتأكد؟** عشان نعرف الرسالة اللي هنـ Grep عليها بالظبط في الـ Intruder.

---

## الخطوة 3 — ابعت الـ Request لـ Intruder

في الـ `POST /my-account/change-password`:

- اضغط بالرايت كليك
- اختار **Send to Intruder**

---

## الخطوة 4 — عدّل الـ Request في Intruder

**غير الـ Request لشكله ده:**

```
username=carlos&current-password=§peter§&new-password-1=abc&new-password-2=xyz
```

**التغييرات:**

```
1) username=wiener  →  username=carlos
   ↑ عشان نهاجم أكونت carlos مش wiener

2) current-password=§peter§
   ↑ حطينا § § على الباسورد عشان Intruder يجرب هنا

3) new-password-1=abc
   new-password-2=xyz
   ↑ مختلفين عشان نتجنب الـ Account Lock
```

---

## الخطوة 5 — حط قائمة الـ Passwords

```
Payloads → Payload type: Simple list
→ الصق قائمة الـ Candidate Passwords من الـ Lab
```

**ليه بنستخدم قائمة جاهزة؟** لأن في الـ Real World، المهاجم بيستخدم قوائم زي `rockyou.txt` فيها ملايين الباسوردات الشائعة.

---

## الخطوة 6 — ضيف Grep Match

```
Settings → Grep Match → Add:
New passwords do not match
```

**ليه بنعمل Grep Match؟**

```
من غير Grep:
لازم تراجع كل Response يدوياً
من بين آلاف الـ Responses 😫

مع Grep:
Burp بيعلم تلقائياً على أي Response
فيه الرسالة دي ✅
```

---

## الخطوة 7 — ابدأ الهجوم

اضغط **Start Attack** وراقب:

```
كل الـ Responses:
"Current password is incorrect" ← باسورد غلط ❌

Response واحد بس:
"New passwords do not match"   ← باسورد صح ✅
```

الـ Response اللي فيه علامة في عمود الـ Grep → ده الباسورد الصح!

---

## الخطوة 8 — عمل Login بـ carlos

```
Username: carlos
Password: الباسورد اللي لقيته
```

روح **My Account** → Lab Solved ✅

---

# 🎯 الخلاصة الذهبية

```
┌─────────────────────────────────────────────┐
│           ليه الثغرة دي خطيرة؟             │
├─────────────────────────────────────────────┤
│                                             │
│  1) رسائل مختلفة = معلومات مسربة           │
│     "do not match" = باسورد صح             │
│                                             │
│  2) Username قابل للتغيير                  │
│     = نقدر نهاجم أي أكونت                 │
│                                             │
│  3) مفيش Rate Limit                        │
│     = نقدر نجرب آلاف الباسوردات           │
│                                             │
│  4) بنتجنب الـ Lock                        │
│     بخلي New Passwords مختلفين             │
│                                             │
└─────────────────────────────────────────────┘
```

---

ابدأ من الخطوة 1 وقولي لو وقفت في أي حتة! 🚀