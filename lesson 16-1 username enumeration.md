

# 🎯 Username Enumeration — من الفكرة للتنفيذ

---

## 📌 الصورة الكبيرة — إنت بتعمل إيه ولماذا؟

قبل ما تبداء تـ **brute force** (تجرب passwords بالقوة) على أي login page — لازم تعرف إجابة سؤال واحد مهم:

> **"الـ username اللي هجرب عليه ده موجود على الـ application ده أصلاً؟"**

لو جربت كذا ألف password على username مش موجود — كل ده وقت ضايع.

الـ **Username Enumeration** (تعداد أو كشف أسماء المستخدمين) هو الخطوة اللي بتجمع فيها **valid usernames** — يعني usernames موجودة فعلاً في الـ database — **قبل ما تبداء الـ brute force**.

هي جزء من مرحلة الـ **Reconnaissance** (الاستطلاع) في الـ web application pentest.

---

## 🧠 الفكرة التقنية — ليه بيحصل Enumeration؟

الـ application لما بيرد على login attempts، ممكن يـ **leak** (يسرّب) معلومات من غير ما يقصد.

السبب الجوهري: الـ **application بيرد بطريقة مختلفة** لما الـ username صح ولما يكون غلط.

**الفرق ده ممكن يظهر في:**

**1 — رسالة الخطأ Error Message:**

- Username غلط: `"No account exists with that username"`
- Username صح + password غلط: `"Incorrect password"`

الفرق ده كفيل يقولك إن الـ username موجود!

التصميم الصح: رسالة موحدة زي `"Invalid username or password"` — مش فرق بين الاتنين.

**2 — Response Time 
(وقت الاستجابة):** لو الـ application بيعمل database query بس لما يلاقي الـ username — الـ response وقتها هيكون أطول شوية. الفرق ممكن يكون milliseconds بس بيكون واضح على scale كبير.

**3 — HTTP Status Code:**
ممكن يرجع `200 OK` لو username موجود و `404 Not Found` لو مش موجود — ده تصميم غلط بردو.

**4 — Response Size
(حجم الـ response):** ممكن الـ page تفضل نفسها بس في الـ HTML فيه فرق بسيط في عدد الـ characters — بيكون واضح لما تقارن الـ responses.

---

## 🗺️ الـ Username Enumeration في الـ Pentest Methodology

```
[Reconnaissance] → [Fingerprinting] → [Enumeration] → [Brute Force] → [Exploitation]
                                           ↑
                                     إحنا هنا دلوقتي
```

الـ **Username Enumeration** بتيجي قبل الـ brute force مباشرة — بعد ما عرفت الموقع بيشتغل إيه وعنده login page.

---

## 🔍 فين بنعمل Enumeration؟

**3 أماكن أساسية:**

**1 — Login Form (صفحة تسجيل الدخول):
** الأكتر شيوعاً — جرب usernames وشوف الـ error messages.

**2 — Registration Form(صفحة التسجيل):** 
لو الـ app بيقول `"Username already taken"` — ده بيقولك إن الـ username موجود! الـ attacker يقدر يجرب قائمة usernames ويشوف أنهي واحد الـ app رفضه.

**3 — Password Reset Form (استعادة الباسورد):
** لو بيقول `"Email not found"` أو `"We've sent an email to that address"` — فيه فرق بردو.

---

## 🛠️ الأدوات والتنفيذ العملي

### الأداة الأساسية: Burp Suite — Intruder

الـ **Burp Suite** (بيرب سويت) هو الأداة الأساسية للـ web application pentesting — بيشتغل كـ **proxy** (وسيط / بروكسي) بين الـ browser بتاعك والـ server — بيمسك كل request وبيخليك تعدل فيه.

**الـ Intruder** (المتطفل) هو الـ module جوه Burp اللي بيعمل automated attacks — بيبعت requests كتير بشكل تلقائي مع تغيير قيمة معينة في كل مرة.

---

### الخطوات العملية خطوة بخطوة:

**الخطوة 1: افتح Burp وشيّل الـ Intercept**

افتح Burp Suite → جوه الـ **Proxy** tab → Intercept is **ON**

اضبط الـ browser يشتغل على الـ Burp proxy (عادةً `127.0.0.1:8080`).

---

**الخطوة 2: امسك الـ Login Request**

روح على الـ login page في الـ browser، اكتب أي username وأي password واضغط login.

الـ request هيتمسك في Burp ويبان شكله كده:

```http
POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

username=testuser&password=testpass
```

---

**الخطوة 3: ابعت الـ Request للـ Intruder**

كليك يمين على الـ request → **Send to Intruder**

---

**الخطوة 4: حدد الـ Position**

روح على **Intruder tab** → **Positions**

هتلاقي الـ Burp حط علامات `§` على كذا حاجة — امسحهم كلهم (Clear §).

بعدين هايلت قيمة الـ username بس وضغط **Add §**:

```
username=§testuser§&password=testpass
```

يعني بس الـ username هو اللي هيتغير في كل request.

**الـ Attack Type:** اختار **Sniper** — بيجرب قيمة واحدة في كل مرة من القائمة.

---

**الخطوة 5: حمّل الـ Payload**

روح على **Payloads tab** → **Payload type: Simple list**

هنا بتحمل قائمة usernames — فيه قوائم جاهزة زي:

- `/usr/share/seclists/Usernames/Names/names.txt`
- أو قائمة من الـ **SecLists** repository على GitHub اللي فيها thousands من common usernames

---

**الخطوة 6: ابداء الهجوم**

اضغط **Start Attack** — Burp هيبداء يبعت requests بالـ usernames من القائمة واحد واحد.

---

**الخطوة 7: حلل الـ Results**

دلوقتي هتبص على الـ results table وتدور على الـ anomaly (الشيء غير الطبيعي) — الـ response اللي مختلف عن البقية.

بتبص على:

- **Length column:** لو response واحد طوله مختلف عن الباقين → ده مشبوه
- **Status column:** لو status code اختلف
- **الـ response نفسه:** كليك على أي response وشوف الـ error message

**مثال عملي:**

- 499 request → response length: 2500 characters
- 1 request → response length: 2600 characters

الـ request اللي طوله 2600 → افتح شوف الـ error message → هتلاقي `"Incorrect password"` بدل `"Invalid username"` → الـ username ده valid!

---

### أداة تانية: ffuf (Fuzz Faster U Fool)

الـ **ffuf** (إف-إف-إيو-إف) هو command-line tool للـ fuzzing — أسرع من Burp Intruder في المجانية.

```bash
ffuf -w /usr/share/seclists/Usernames/Names/names.txt \
     -X POST \
     -d "username=FUZZ&password=testpass" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://target.com/login \
     -mr "Invalid password"
```

**شرح الأوبشنز:**

- `-w` (wordlist — قائمة الكلمات): مسار قائمة الـ usernames
- `-X POST`: نوع الـ request
- `-d` (data — البيانات): الـ body بتاع الـ request — `FUZZ` هي الكلمة اللي هتتبدل
- `-H` (header — ترويسة الطلب): بنحط الـ Content-Type
- `-u` (url — الرابط): الـ target
- `-mr` (match regex — مطابقة النص): هيظهر بس الـ results اللي فيها الجملة دي في الـ response

---

## 🧪 مثال كامل على سيناريو حقيقي

**السيناريو:** عندك login page على `http://target.com/login`

بتجرب يدوياً:

- username: `admin` + password: `wrongpass` → رسالة: `"Invalid password"`
- username: `notexist` + password: `wrongpass` → رسالة: `"No account found"`

**الرسالتين مختلفتين ← الـ application vulnerable لـ Username Enumeration!**

دلوقتي:

1. بتمسك الـ POST request في Burp
2. بتبعته للـ Intruder
3. بتحمل قائمة usernames كـ payload
4. بتفلتر النتايج على الـ response length أو الـ error message
5. بتجمع قائمة بالـ **valid usernames**

في الخطوة الجاية: الـ **Brute Force** على الـ usernames دي بقى.

---

## 🔒 الدفاع — عشان تفهم الثغرة من جوا

الـ developer الكويس بيعمل إيه عشان يمنع الـ enumeration؟

1. **Generic Error Messages:** رسالة موحدة `"Invalid username or password"` للحالتين
2. **Consistent Response Time:** يخلي الـ response time متعادل حتى لو الـ username مش موجود
3. **Rate Limiting:** يحدد عدد المحاولات من نفس الـ IP
4. **Account Lockout:** يـ lock الـ account بعد كذا محاولة فاشلة
5. **CAPTCHA** بعد كذا محاولة

---

## 📋 مصطلحات الدرس في سياقها

|English Term|المعنى|النطق|
|---|---|---|
|Username Enumeration|كشف / تعداد أسماء المستخدمين|يوزرنيم إينيوميريشن|
|Brute Force|هجوم القوة الغاشمة — تجريب كل الاحتمالات|بروت فورس|
|Reconnaissance|مرحلة الاستطلاع وجمع المعلومات|ريكونيسانس|
|Proxy|وسيط — بيتوسط بين الـ browser والـ server|بروكسي|
|Intruder|أداة الـ automated attacks في Burp|إنترودر|
|Fuzzing|تجريب قيم كتير تلقائياً على input|فازينج|
|Payload|القيمة أو البيانات اللي بنبعتها في الهجوم|بيلود|
|Wordlist|قائمة كلمات / مسارات / usernames للهجوم|ووردليست|
|Anomaly|الشيء غير الطبيعي في الـ results|أنومالي|
|Response Length|طول / حجم الـ response بالـ bytes|ريسبونس لينث|
|Status Code|كود حالة الـ HTTP response|ستاتس كود|
|Rate Limiting|تحديد عدد الطلبات في وقت معين|ريت ليميتنج|
|Account Lockout|قفل الحساب بعد محاولات فاشلة|أكاونت لوكاوت|
|SecLists|مجموعة قوائم جاهزة للـ pentesting|سيك-ليستس|
|ffuf|أداة fuzzing سريعة من الـ command line|إف-إف-إيو-إف|
|Generic Error|رسالة خطأ عامة مش بتكشف تفاصيل|جينيريك إيرور|

---

## ✅ ملخص الفكرة العملية

> قبل ما تبداء أي brute force — لازم تعرف الـ valid usernames. الـ Username Enumeration هي الخطوة اللي بتكشف لك مين موجود على الـ application. بتدور على أي **فرق** في الـ response — سواء في الـ error message أو الـ response length أو الـ status code — وبتستخدم Burp Intruder أو ffuf عشان تعمل الموضوع تلقائي على قائمة usernames كبيرة.

---

**جاهز تكمل على نقطة تانية؟** الخطوة الطبيعية الجاية هي **Brute Force على الـ Login** بعد ما عندك الـ valid usernames. ولا عندك سؤال على حاجة في الدرس ده الأول؟

\