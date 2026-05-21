بالظبط!

**Upload Directory Listing** 
معناها إن الـ server مش مخبي محتوى الـ `/wp-content/uploads/` — فأي حد يكتب الـ URL ده في المتصفح يقدر يشوف كل الملفات والفولدرات جوا من غير ما يكون عنده أي صلاحية.

ده نوع ثغرة اسمه **Directory Listing** أو **Directory Browsing** — وهو في حد ذاته **finding مهم** في أي pentest report لأنه:

|الخطر|السبب|
|---|---|
|كشف ملفات حساسة|زي الـ logs اللي لاقيناها|
|كشف هيكل الموقع|بيساعد المهاجم يفهم الموقع|
|تسريب بيانات|صور، ملفات، backups ممكن تكون مكشوفة|

---

دلوقتي افتح الـ `failed-logins/` وقولي إيه اللي جواه 🎯
gobuster 
	dirsearch  fuzz
	صح! لو الموقع مش WordPress، هنستخدم الـ **Web Application Pentesting Methodology** العامة.
	
	ممتاز! الـ output كامل. خلينا نحلله.

---

## تحليل الـ WPScan Output

---

### اللي لاقيناه:

|المعلومة|القيمة|
|---|---|
|WordPress Version|4.4|
|User|`pentester`|
|Plugins|❌ مفيش plugins|
|XML-RPC|✅ مفعّل|
|Upload Directory Listing|✅ مفعّل|
|CVEs على الـ Core|🔴 **116 ثغرة!**|

---

### الملاحظة المهمة:

مفيش plugins — يعني مسار الـ adrotate اللي عملناه في Lab 1 مش موجود هنا.

بس الـ API Token جابلنا **116 CVE** على الـ WordPress Core نفسه — ده كنز!

---

## إيه الخطوة الجاية؟

بما إن الهدف هو **ندخل بثغرة تانية** غير الـ brute force — عندنا خيارين مهمين من الـ CVEs:

**خيار A — RCE في PHPMailer:**

```
WordPress 4.3-4.7 - Remote Code Execution (RCE) in PHPMailer
Metasploit module: exploit/unix/webapp/wp_phpmailer_host_header
```

**خيار B — Authenticated Code Execution:**

```
WordPress 3.7-5.0 - Authenticated Code Execution
CVE-2019-8942
Metasploit module: exploit/multi/http/wp_crop_rce
```

---

إيه اللي الدكتور قالك تستخدمه؟ ولا تحب نجرب واحدة منهم؟ 🎯


ممتاز! دلوقتي فاهمين الثغرة. خلينا نشرحها الأول وبعدين نشغّلها.

---

## الثغرة إيه بالظبط؟

الـ plugin ده بيسمح للمستخدم يرفع ملفات — صور، فيديوهات، إلخ. المشكلة إن فيه حقل اسمه `accept_file_types` المفروض يحدد الامتدادات المسموحة — بس السيرفر **مش بيتحقق منه على السيرفر نفسه**، بيصدّقه من الـ request مباشرةً.

يعني إحنا نقدر نغيّر القيمة دي ونضيف `php` ونرفع **web shell** كامل!

---

## الخطة:

**الخطوة ١:** نروح لصفحة الـ plugin في الموقع ونحاول نرفع ملف **الخطوة ٢:** نستخدم **Burp Suite** عشان نعترض الـ request ونعدّله **الخطوة ٣:** نضيف `php` للـ `accept_file_types` ونغيّر اسم الملف لـ `.php` **الخطوة ٤:** نكتب web shell جوا الملف **الخطوة ٥:** نلاقي الملف في `/wp-content/uploads/public/wtf-fu_files/default/`

---

## الخطوة الأولى: نلاقي صفحة الـ plugin

افتح في المتصفح:

```
https://8by2plz674jgz0cd9k11wpkq8.ap-south-18.attackdefensecloudlabs.com/
```

دور على أي صفحة فيها **upload form** أو زرار رفع ملفات — أو جرب:

```
https://8by2plz674jgz0cd9k11wpkq8.ap-south-18.attackdefensecloudlabs.com/?page_id=2
```

قولي إيه اللي شايفه 🎯
وشوف في أي صفحة فيها الـ upload form — غالباً هتلاقي page اسمها **Upload** أو **WTF** أو حاجة مشابهة.

قولي إيه الصفحات الموجودة 🎯فيه page واحدة اسمها **Sample Page** — اضغط عليها بالماوس واختار **View** عشان تشوفها في المتصفح، وقولي إيه اللي فيها 🎯



لاحظ إن الـ URL اتغير — ده lab تاني غير الأول!

الـ lab الأول كان:

```
8by2plz674jgz0cd9k11wpkq8.ap-south-18
```

الدلوقتي:

```
gd14t906vwv6ou0gyg0rpibmp.ap-south-17
```

---

اضغط على **View Page** في الأعلى عشان تشوف الصفحة في المتصفح، وقولي إيه اللي فيها 🎯
ده مش الـ upload بتاع الـ plugin — ده الـ WordPress Media Upload العادي، ده مش هينفع معانا.

اغلق الـ popup ده واضغط على **View Page** في الأعلى عشان تشوف الصفحة الفعلية في المتصفح 🎯


الصفحة دي مفيهاش الـ upload form بتاع الـ plugin.

محتاج نضيف الـ plugin shortcode على صفحة — روح الـ wp-admin وافتح الـ Sample Page للتعديل:

```
Pages → Sample Page → Edit
```

وبعدين في الـ Text editor اكتب:

```
[wtf-fu-uploader]
```

وبعدين اضغط **Update** وافتح الصفحة تاني 🎯

اضغط على **Text** (مش Visual) عشان تكتب الـ shortcode:

اضغط تبويب **Text** في الأعلى يمين الـ editor، وبعدين في أول السطر اكتب:

```
[wtf-fu-uploader]
```

وبعدين اضغط **Update** 🎯
ممتاز! شايف `[wtf-fu-uploader]` في أول السطر — تمام!

دلوقتي اضغط **Update** وبعدين اضغط **View page** عشان تشوف الصفحة 🎯

## إيه اللي بيعمله `[wtf-fu-uploader]`؟

ده اسمه **Shortcode** (شورت-كود) — ده كود مختصر بيحوله WordPress لـ HTML كامل على الصفحة.

لما بتحط `[wtf-fu-uploader]` في الصفحة، الـ plugin بيحوله لـ **form رفع ملفات** كامل على الصفحة — بيظهر للزوار زرار upload.

---

دلوقتي اضغط **Update** وبعدين **View page** وهتلاقي الـ upload form ظهر على الصفحة.

بعد ما تشوفه، هنبدأ نشغّل **Burp Suite** عشان نعترض الـ request ونعدّله 🎯