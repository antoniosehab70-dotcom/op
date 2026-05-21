
# 🎯 Brute Force على الـ Login — من الفكرة للتنفيذ

---

## 📌 الصورة الكبيرة — إنت بتعمل إيه ولماذا؟

عندك دلوقتي قائمة بالـ **valid usernames** من الخطوة اللي فاتت.

السؤال الجاي: **إيه الـ password بتاع كل user؟**

الـ **Brute Force Attack** (هجوم القوة الغاشمة) هو إنك بتجرب passwords كتير تلقائياً على username محدد عشان تلاقي الـ password الصح.

بس مش كل brute force بيتعمل بنفس الطريقة — فيه فرق مهم بين الأنواع:

---

## 🧠 أنواع الـ Password Attacks

### 1 — Brute Force خالص

بتجرب **كل الاحتمالات الممكنة** — حروف وأرقام ورموز من أول `a` لـ `zzzzzzzzz`.

**المشكلة:** لو الـ password طويل — هياخد سنين حرفياً. مش عملي إلا لو passwords قصيرة جداً (4-6 characters).

---

### 2 — Dictionary Attack (هجوم القاموس)

بتجرب passwords من **قائمة جاهزة** — passwords شايعة الناس بتستخدمها فعلاً زي `password123` و `admin` و `letmein`.

**ليه بينجح؟** لأن دراسات بتقول إن الأغلبية بتستخدم passwords ضعيفة ومتوقعة.

أشهر قائمة: **rockyou.txt** — فيها 14 مليون password من leak حقيقي.

---

### 3 — Credential Stuffing (حشو البيانات المسربة)

بتاخد **username:password combinations** من data breaches (تسريبات بيانات) حقيقية وبتجربها على مواقع تانية.

**ليه بينجح؟** لأن ناس كتير بتستخدم نفس الـ password على مواقع مختلفة.

---

### 4 — Password Spraying (رش الباسورد)

بدل ما تجرب passwords كتير على user واحد — بتجرب **password واحد على users كتير**.

**ليه مهم؟** عشان يتجنب الـ **Account Lockout** (قفل الحساب) — لو كل user اتجرب عليه password واحد بس، مش هيتـ lock.

---

## 🗺️ في الـ Pentest Methodology — جاي بعد إيه؟

```
[Username Enumeration] → [Password Attack] → [Access] → [Exploitation]
         ✅ خلصنا              ↑ إحنا هنا
```

الـ Brute Force هو الخطوة الطبيعية بعد الـ Enumeration — عندك الـ usernames، دلوقتي تجيب الـ passwords.

---

## 🛠️ التنفيذ العملي بـ Burp Suite Intruder

### السيناريو:

عندك login page وعندك valid username: `admin` عايز تجرب قائمة passwords عليه.

---

**الخطوة 1: امسك الـ Login Request في Burp**

```http
POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=testpass
```

---

**الخطوة 2: ابعت للـ Intruder**

كليك يمين → **Send to Intruder**

---

**الخطوة 3: حدد الـ Position**

في الـ **Positions tab** — امسح كل الـ `§` وحط الـ position على الـ password بس:

```
username=admin&password=§testpass§
```

**Attack Type: Sniper** — بيجرب كل password من القائمة واحد واحد.

---

**الخطوة 4: حمّل الـ Payload**

**Payloads tab** → **Simple list** → **Load** → اختار الـ wordlist

```bash
# أشهر القوائم:
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
```

---

**الخطوة 5: Start Attack وحلل النتايج**

بص على:

- **Length:** الـ response اللي طوله مختلف → ده محتمل يكون الـ password الصح
- **Status Code:** لو جاب `302 Found` بدل `200 OK` → ده redirect بعد login ناجح
- **الـ response نفسه:** دور على `"Welcome"` أو `"Dashboard"` أو أي علامة إن الـ login نجح

---

## 🛠️ التنفيذ بـ Hydra

الـ **Hydra** (هايدرا) هو command-line tool متخصص في الـ password attacks على بروتوكولات كتير — HTTP, FTP, SSH, RDP وغيرهم.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid password"
```

**شرح الأوبشنز:**

- `-l admin` : الـ username المحدد (حرف L صغير)
- `-L users.txt` : لو عندك قائمة usernames كمان
- `-P rockyou.txt` : قائمة الـ passwords
- `http-post-form` : نوع الـ attack — POST request على HTTP
- `/login` : الـ path بتاع الـ login
- `^USER^` : Hydra هيحط الـ username هنا
- `^PASS^` : Hydra هيحط الـ password هنا
- `Invalid password` : الـ **failure string** — الجملة اللي بتظهر لما الـ login يفشل — Hydra هيعرف إن الـ password غلط لما يلاقي الجملة دي في الـ response

---

### لو عندك قائمة usernames وقائمة passwords:

```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt target.com http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid password"
```

---

## 🛠️ التنفيذ بـ ffuf

```bash
ffuf -w /usr/share/wordlists/rockyou.txt:PASS \
     -X POST \
     -d "username=admin&password=PASS" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://target.com/login \
     -mr "Welcome" \
     -t 50
```

- `-mr "Welcome"` : اعرضلي بس الـ requests اللي في الـ response بتاعها كلمة `Welcome`
- `-t 50` : الـ threads (خيوط التنفيذ المتوازي) — 50 request في نفس الوقت

---

## ⚙️ Cluster Bomb Attack — Username + Password في نفس الوقت

لو عندك قائمة usernames **وقائمة passwords** وعايز تجربهم مع بعض — بتستخدم **Cluster Bomb** في Burp Intruder.

**الفرق:**

- **Sniper:** position واحدة بتتغير
- **Cluster Bomb:** أكتر من position بتتغيروا مع بعض — بيجرب كل combination ممكنة

```
username=§admin§&password=§testpass§
```

- **Payload Set 1:** قائمة usernames
- **Payload Set 2:** قائمة passwords

هيجرب كل username مع كل password — لو عندك 100 user و 100 password → 10,000 request.

> ⚠️ **تحذير عملي:** الـ Cluster Bomb بيبعت requests كتير جداً — ممكن يبطّأ الـ server أو يلفت الانتباه. استخدمه بحكمة في الـ real pentests.

---

## 🚧 الحواجز اللي ممكن تواجهها وإزاي تتعامل معاها

### 1 — Account Lockout (قفل الحساب)

لو الـ app بيـ lock الحساب بعد 5 محاولات فاشلة مثلاً.

**الحل:** استخدم **Password Spraying** — جرب password واحد على كل الـ users بدل ما تركز على user واحد.

---

### 2 — Rate Limiting (تحديد معدل الطلبات)

الـ server بيحظر الـ IP بعد requests كتير في وقت قصير.

**الحلول:**

- قلل الـ threads وزود الـ delay بين الـ requests
- في Burp: **Intruder → Options → Throttling** — حط delay بين كل request
- في ffuf: `-rate 10` (10 requests في الثانية بس)

---

### 3 — CAPTCHA

بيظهر بعد محاولات معينة.

**الحل:** لازم تعمل CAPTCHA bypass الأول — وده موضوع لوحده اتكلمنا عليه في المحاضرة.

---

## 🧪 مثال كامل — سيناريو من البداية للنهاية

**السيناريو:** عندك `http://target.com/login`

**خطوة 1 — Username Enumeration:** لقيت إن الـ app بيرد بـ `"Invalid password"` لما username صح وبيرد بـ `"No account found"` لما غلط.

استخدمت Burp Intruder مع wordlist → لقيت valid username: `john`

**خطوة 2 — Password Attack:**

```bash
hydra -l john -P /usr/share/wordlists/rockyou.txt target.com http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid password"
```

**النتيجة:**

```
[80][http-post-form] host: target.com   login: john   password: password123
```

**خطوة 3 — Login:** استخدمت `john:password123` → دخلت على الـ account.

---

## 🔒 الدفاع — عشان تفهم الثغرة من جوا

|الثغرة|الحل|
|---|---|
|مفيش Account Lockout|بعد 5 محاولات → lock لمدة معينة|
|مفيش Rate Limiting|حظر الـ IP بعد requests كتير|
|Weak Passwords مسموح بيها|فرض password policy قوية|
|Error messages مختلفة|generic message موحدة|
|مفيش CAPTCHA|إضافة CAPTCHA بعد كذا محاولة|
|مفيش MFA|إضافة 2FA طبقة تانية من الحماية|

---

## 📋 مصطلحات الدرس في سياقها

|English Term|المعنى|النطق|
|---|---|---|
|Dictionary Attack|هجوم القاموس — passwords من قائمة جاهزة|ديكشنري أتاك|
|Credential Stuffing|حشو بيانات مسربة من breaches على مواقع تانية|كريدنشال ستافنج|
|Password Spraying|رش password واحد على users كتير|باسورد سبريينج|
|Data Breach|تسريب بيانات حقيقية|داتا بريتش|
|rockyou.txt|قائمة passwords من تسريب حقيقي|روك-يو|
|Wordlist|قائمة كلمات للهجوم|ووردليست|
|Hydra|أداة password attack للـ command line|هايدرا|
|Threads|خيوط التنفيذ المتوازي — كام request في نفس الوقت|ثريدز|
|Failure String|الجملة اللي بتدل إن الـ login فشل|فيليور سترينج|
|Cluster Bomb|نوع هجوم في Burp بيجرب كل combinations|كلاستر بومب|
|Sniper|نوع هجوم في Burp بيغير position واحدة|سنايبر|
|Throttling|تبطيء الطلبات عمداً عشان تتجنب الحظر|ثروتلنج|
|Redirect|إعادة توجيه — علامة على login ناجح|ريدايريكت|
|Account Lockout|قفل الحساب بعد محاولات فاشلة|أكاونت لوكاوت|
|Rate Limiting|تحديد عدد الطلبات في وقت معين|ريت ليميتنج|

---

## ✅ ملخص الفكرة العملية

> الـ Brute Force مش بس "جرب passwords كتير" — ده علم فيه اختيار صح للأداة والقائمة والنوع. عارف الـ target بيـ lock accounts → استخدم Password Spraying. مفيش lockout → Dictionary Attack بـ Hydra أو Burp. عندك users كتير وpasswords كتير → Cluster Bomb. الفكرة كلها في إنك تبحث عن الـ **anomaly** في الـ response — الـ response المختلف عن الباقيين ده هو الـ valid credential.

---

**الخطوة الجاية الطبيعية هي الـ CAPTCHA Bypass** — عشان لو الـ app عنده CAPTCHA تعرف تعداه وتكمل الـ brute force.

نكمل؟

