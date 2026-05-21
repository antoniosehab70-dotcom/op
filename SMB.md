ممتاز يا أنطونيوس 👏 واضح إنك بدأت تفهم **SMB فعلاً** لأنك استخدمته في:

- عرض الـ shares
    
- تحميل الملفات (`get`)
    
- رفع الملفات (`put`)
    
- استغلال **EternalBlue**
    

خلّينا نعمل **شرح كامل عملي لـ SMB** مثل ما يستخدمه البينتستر الحقيقي.

---

# أولاً ما هو SMB

SMB اختصار:

```text
Server Message Block
```

وهو بروتوكول مشاركة الملفات في أنظمة:

Windows

يسمح بـ:

- مشاركة الملفات
    
- مشاركة الطابعات
    
- Remote administration
    

---

# البورتات التي يعمل عليها

|Port|الاستخدام|
|---|---|
|139|NetBIOS SMB|
|445|SMB Direct|

غالباً ستراه في nmap:

```bash
445/tcp open  microsoft-ds
139/tcp open  netbios-ssn
```

---

# أول شيء يفعله البينتستر

## 1️⃣ فحص SMB

باستخدام:

Nmap

```bash
nmap -p 139,445 --script smb* IP
```

هذا السكربت يظهر:

- shares
    
- version
    
- vulnerabilities
    

---

# ثانياً Enumeration

أهم مرحلة في SMB.

## عرض الـ shares

باستخدام:

```bash
smbclient -L //IP
```

مثال:

```bash
smbclient -L //10.10.10.10
```

لو طلب password يمكنك الضغط:

```text
Enter
```

أحياناً يسمح بالدخول بدون password.

---

# الدخول بدون كلمة مرور

إذا كان السيرفر يسمح بـ:

```text
Anonymous access
```

نستخدم:

```bash
smbclient -L //IP -N
```

`-N` يعني:

```text
no password
```

---

# الدخول إلى share

لو ظهر:

```text
public
backup
files
```

ندخل مثلاً:

```bash
smbclient //IP/public
```

---

# الدخول بيوزر

```bash
smbclient //IP/public -U username
```

مثال:

```bash
smbclient //10.10.10.10/public -U admin
```

---

# الأوامر داخل SMB

بعد الدخول سيظهر:

```text
smb: \>
```

---

## عرض الملفات

```bash
ls
```

---

## الدخول إلى مجلد

```bash
cd folder
```

---

## تحميل ملف

```bash
get file.txt
```

ينزل الملف على جهازك.

---

## رفع ملف

```bash
put shell.php
```

يرفع الملف إلى السيرفر.

---

## تحميل مجلد كامل

```bash
mget *
```

---

## الخروج

```bash
exit
```

---

# أدوات مهمة لـ SMB

## 1️⃣ enumeration قوي

أداة:

enum4linux

```bash
enum4linux -a IP
```

تظهر:

- users
    
- shares
    
- policies
    

---

## 2️⃣ brute force

أداة:

Hydra

```bash
hydra -L users.txt -P passwords.txt smb://IP
```

---

## 3️⃣ exploitation

أداة:

Metasploit Framework

مثال:

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

---

# استغلال EternalBlue

الثغرة:

EternalBlue

---

### خطواتها

1️⃣ فتح metasploit

```bash
msfconsole
```

---

2️⃣ اختيار exploit

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

---

3️⃣ تحديد الهدف

```bash
set RHOSTS IP
```

---

4️⃣ تشغيل الهجوم

```bash
run
```

---

# أداة مهمة جداً للهجوم على SMB

الأداة التي جاء عنها السؤال في الامتحان:

CrackMapExec

---

### استخدامها

فحص SMB:

```bash
crackmapexec smb IP
```

---

### تجربة credentials

```bash
crackmapexec smb IP -u admin -p password
```

---

### تجربة hashes

```bash
crackmapexec smb IP -u admin -H NTLMHASH
```

---

# إجابة أسئلة الامتحان

## السؤال الأول

> Which SMB security feature helps prevent SMB relay attacks?

الإجابة الصحيحة:

```text
SMB Signing
```

لأنه يضيف:

```text
message authentication
```

ويمنع:

```text
SMB relay attacks
```

---

## السؤال الثاني

> Which tool is commonly used for SMB exploitation?

الإجابة:

```text
CrackMapExec
```

لأنه يستخدم في:

- SMB attacks
    
- lateral movement
    
- password spraying
    

---

# أخطر 4 مشاكل في SMB يستغلها البينتستر

### 1️⃣ Anonymous shares

يمكن الدخول بدون password.

---

### 2️⃣ Writable shares

يمكن رفع ملفات.

---

### 3️⃣ Weak credentials

passwords ضعيفة.

---

### 4️⃣ EternalBlue

ثغرة في Windows.

---

# كيف يفكر البينتستر عندما يرى SMB

عندما ترى في nmap:

```text
445 open smb
```

اسأل نفسك:

1️⃣ هل يمكن الدخول anonymous  
2️⃣ هل هناك shares  
3️⃣ هل يمكن تحميل ملفات  
4️⃣ هل يمكن رفع shell  
5️⃣ هل هناك vulnerability

---

# أهم أوامر SMB التي يجب أن تحفظها

```bash
smbclient -L //IP
```

عرض الـ shares.

---

```bash
smbclient //IP/share
```

الدخول إلى share.

---

```bash
ls
```

عرض الملفات.

---

```bash
get file
```

تحميل ملف.

---

```bash
put file
```

رفع ملف.

---

لو أردت، في الرسالة القادمة أشرح لك **أخطر 10 بورتات في البينتست وكيف يفكر المخترق عند رؤيتها**، وهذا سيغير طريقة حلك للرومات تمامًا.|Port|Service|ماذا تفعل|
|---|---|---|
|21|FTP|محاولة anonymous login|
|22|SSH|تجربة credentials|
|25|SMTP|user enumeration|
|53|DNS|zone transfer|
|80|HTTP|web exploitation|
|139/445|SMB|shares enumeration|
|3306|MySQL|database access|
|3389|RDP|remote login|