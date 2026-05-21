dig 
a m ass
the harvester 
hims.edu.eg

جامد جدًا 👌🔥  
إنت دلوقتي دخلت رسمي في **Passive Recon**.

الدكتور اداكم أدوات، وعايزكم تفهموا:

- كل أداة بتجيب إيه
    
- أستخدمها إمتى
    
- وأجمع منها معلومات عن Target زي:  
    `hims.edu.eg`
    

خليني أرتبهم لك 👇

---

# 🎯 أولًا: Passive Recon يعني إيه؟

يعني:

> تجمع معلومات عن الهدف  
> من غير ما تتفاعل مع السيرفر نفسه مباشرة

يعني مفيش Scan  
مفيش SYN  
مفيش لمس مباشر

---

# 1️⃣ dig

🔹 dig = DNS lookup tool  
بتجيب معلومات DNS.

مثال:

```
dig hims.edu.eg
```

هيجيب:

- IP Address
    
- Name Servers
    
- MX Records
    

---

## أوامر مهمة

```
dig hims.edu.eg A
```

يجيب الـ IP

```
dig hims.edu.eg MX
```

يجيب Mail Servers

```
dig hims.edu.eg NS
```

يجيب Name Servers

---

🎯 ليه مهم؟

- تعرف السيرفرات المرتبطة بالدومين
    
- ممكن تلاقي Subdomains
    
- تعرف البريد الإلكتروني بيعدي على فين
    

ده كله Passive.

---

# 2️⃣ nslookup

نفس فكرة dig تقريبًا  
أداة DNS قديمة شوية.

```
nslookup hims.edu.eg
```

ممكن تستخدمها في Reverse Lookup:

```
nslookup IP
```

---

# 3️⃣ whois

مهم جدًا 👌🔥

```
whois hims.edu.eg
```

بيجيب:

- اسم الجهة المسجلة
    
- Email
    
- DNS servers
    
- Registrar
    

🎯 ده بيساعد في:

- جمع Email
    
- Social Engineering
    
- معرفة شركة الاستضافة
    

---

# 4️⃣ theHarvester

دي أداة قوية جدًا.

بتجمع:

- Emails
    
- Subdomains
    
- Hosts
    
- IPs
    

من مصادر زي:

- Google
    
- Bing
    
- LinkedIn
    
- DNS
    

مثال:

```
theHarvester -d hims.edu.eg -b google
```

🎯 ده بيجمع معلومات من Google عن الدومين.

---

# ليه ده مهم؟

لو لقيت:

[admin@hims.edu.eg](mailto:admin@hims.edu.eg)  
[it@hims.edu.eg](mailto:it@hims.edu.eg)

دي معلومات ذهب.

ممكن تستخدمها بعدين في:

- Phishing
    
- Password spraying
    

---

# نربط ده بخطوات PenTest

ده كله في مرحلة:

Reconnaissance

وبالتحديد:

Passive Information Gathering

---

# دلوقتي تعمل إيه عمليًا؟

1️⃣ اختار Target  
مثلاً: hims.edu.eg

2️⃣ اعمل:

- whois
    
- dig
    
- nslookup
    
- theHarvester
    

3️⃣ اكتب في كشكولك:

- IP
    
- Mail Server
    
- Subdomains
    
- Emails
    
- Registrar
    

4️⃣ اسأل نفسك:

- الشركة مستضافة فين؟
    
- عندهم Email ظاهر؟
    
- في Subdomain شكله Development؟
    

---

# الفرق بين Passive و Active هنا

Passive:  
whois  
theHarvester  
Google

Active:  
nmap  
ping  
port scan

---

# المصطلحات اللي تثبت

DNS Records  
MX Record  
NS Record  
A Record  
Subdomain Enumeration  
Open Source Intelligence (OSINT)

---

# نقطة مهمة جدًا

Passive Recon أحيانًا يجيب معلومات أخطر من Scan.

ليه؟

لأن الموظفين ساعات ينشروا بيانات بنفسهم 😄

---

# سؤال يخليك تفكر

لو لقيت Subdomain اسمه:

dev.hims.edu.eg

ده يخليك تفكر في إيه؟ 😈