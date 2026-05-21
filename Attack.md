
هنبدأ بالأساسيات الثقيلة اللي فعليًا بتستخدمها يوميًا في AD pentest.  
نبدأ بـ **NTLM + SMB + Responder + NTLM Relay** لأن ده عمود هجمات الشبكات الداخلية.

---

# 🔴 أولاً: نفهم NTLM قبل أي أداة

## NTLM يعني إيه؟

NTLM = New Technology LAN Manager  
ده بروتوكول Authentication قديم في Windows.

بيشتغل بطريقة اسمها:  
Challenge – Response Authentication

يعني إيه الكلام ده عمليًا؟

1. العميل يقول للسيرفر: عايز أدخل
    
2. السيرفر يبعته رقم عشوائي اسمه Challenge
    
3. العميل ياخد الـ Challenge ده + الهاش بتاع الباسورد
    
4. يعمل عملية تشفير ويرجع Response
    
5. السيرفر يقارن النتيجة باللي عنده
    

⚠️ الباسورد نفسه مش بيتبعت  
لكن الهاش بيتستخدم في الحساب.

---

# 🔵 SMB بقى فين هنا؟

SMB = Server Message Block  
بروتوكول مشاركة ملفات في Windows  
بيشتغل على TCP 445

لما تعمل:

```
\\192.168.1.10\share
```

أنت بتعمل SMB Authentication

لو Kerberos مش شغال  
SMB بيرجع يستخدم NTLM

---

# 🟡 الهجوم الأول: LLMNR / NBT-NS Poisoning

## البروتوكولات دي إيه؟

LLMNR = Link-Local Multicast Name Resolution  
بورت UDP 5355

NBT-NS = NetBIOS Name Service  
بورت UDP 137

وظيفتهم:  
لو الجهاز مش لاقي اسم في DNS  
يسأل الشبكة كلها:  
"مين اسمه FILESERVER؟"

أي جهاز يقدر يرد.

---

## هنا بتيجي أداة Responder

Responder بتعمل إيه؟

1. تقعد تستنى LLMNR / NBT-NS requests
    
2. لما حد يسأل عن اسم
    
3. ترد تقول: أنا هو
    
4. الجهاز يحاول يعمل NTLM authentication معاك
    
5. Responder يسجل NTLM hash
    

---

## تشغيل Responder

```
responder -I eth0
```

-I يعني interface

Responder بيعمل:

- يسمع على UDP 5355
    
- يسمع على UDP 137
    
- يعمل Fake SMB server
    
- يستقبل NTLM challenge/response
    
- يحفظها في ملف
    

---

## ليه ده خطير؟

لأنك حصلت على NTLM Hash  
مش الباسورد  
لكن الهاش ممكن:

- يتكسر Offline
    
- أو يتستخدم في Pass-the-Hash
    
- أو يتعمل بيه NTLM Relay
    

---

# 🔴 الهجوم الثاني: NTLM Relay

ده أقوى من مجرد التقاط الهاش.

## الفكرة:

بدل ما تكسر الهاش  
تعيد تمريره مباشرة لسيرفر تاني

---

## إزاي بيحصل عمليًا؟

1. Responder يخلي جهاز يعمل NTLM auth
    
2. بدل ما يخزن الهاش
    
3. يحوله فورًا لسيرفر هدف
    
4. السيرفر الهدف يقبل المصادقة
    

---

## الأداة: ntlmrelayx.py (من Impacket)

Impacket عبارة عن:  
مجموعة سكربتات Python للتعامل مع بروتوكولات Windows  
(مش مجرد أداة واحدة)

---

## تشغيل NTLM Relay

```
ntlmrelayx.py -t smb://192.168.1.20
```

-t = target

الأداة بتعمل داخليًا:

- تعمل Fake SMB listener
    
- تستقبل NTLM challenge/response
    
- تعيد تمريره للهدف
    
- لو نجح:
    
    - تفتح لك shell
        
    - أو تضيف user
        
    - أو تكتب ملف
        

---

## إمتى الهجوم يفشل؟

لو:

- SMB Signing مفعل
    
- الهدف بيستخدم Kerberos بس
    
- Extended Protection مفعل
    

---

# 🟢 Pass-the-Hash

دلوقتي حصلت على NTLM hash

NTLM hash هو:  
MD4(password)

مش متملح  
يعني نفس الباسورد = نفس الهاش

---

## إزاي تستخدمه؟

أداة netexec (CrackMapExec سابقًا)

```
netexec smb 192.168.1.50 -u administrator -H NTLMHASH
```

-H معناها Hash مش Password

الأداة بتعمل إيه داخليًا؟

- تبعت NTLM authentication request
    
- تستخدم الهاش بدل الباسورد
    
- تكمل handshake
    

لو الحساب Admin  
تقدر تنفذ أوامر.

---

# 🟣 Kerberoasting بالتفصيل

## Kerberos بيشتغل إزاي؟

1. User ياخد TGT من KDC
    
2. يطلب Service Ticket لخدمة معينة
    
3. KDC يبعت TGS متشفر بباسورد الحساب الخدمي
    

هنا الثغرة:

أي user عادي يقدر يطلب Service Ticket  
والتذكرة دي متشفرة بباسورد الخدمة

---

## الأداة: GetUserSPNs.py

```
GetUserSPNs.py domain/user:pass -request
```

الأداة بتعمل:

- LDAP query تجيب الحسابات اللي عندها SPN
    
- تطلب TGS من KDC
    
- تحفظ التذكرة بصيغة crackable
    

---

## بعد كده

تكسر التذكرة بـ Hashcat:

```
hashcat -m 13100 hash.txt wordlist.txt
```

لو الباسورد ضعيف → تكسب Service Account  
وغالبًا بيبقى له صلاحيات عالية.

---

# 🟡 AS-REP Roasting

في Kerberos عادةً لازم Pre-Authentication

لكن لو الحساب متعلم عليه:  
"Does not require pre-authentication"

تقدر تطلب TGT بدون باسورد

---

## الأداة:

```
GetNPUsers.py domain/ -usersfile users.txt -no-pass
```

الأداة:

- تبعت AS-REQ بدون PreAuth
    
- لو الحساب ضعيف
    
- KDC يبعت TGT متشفر بباسورده
    
- تكسره Offline
    

---

# 🧠 ليه كل ده مهم؟

لأن معظم اختراقات الشركات داخليًا بتمشي كده:

LLMNR → NTLM hash  
↓  
NTLM Relay  
↓  
User Access  
↓  
Kerberoast  
↓  
Priv Esc  
↓  
Domain Admin

---
    

دلوقتي هنكمّل باقي الهجمات المهمة في بيئة Active Directory والـ Internal Network.

---

# 🔴 1) LDAP Attacks

## الأول: LDAP يعني إيه؟

LDAP = Lightweight Directory Access Protocol  
بروتوكول للتعامل مع Active Directory.

بيشتغل على:

- TCP 389 (LDAP)
    
- TCP 636 (LDAPS مشفر)
    

هو مش بروتوكول مصادقة بس،  
ده بروتوكول استعلام وتعديل بيانات.

يعني:

- تجيب Users
    
- تجيب Groups
    
- تعدل Attributes
    
- تضيف User
    

---

## الهجوم 1: LDAP Enumeration

### الفكرة

لو عندك:

- User عادي  
    أو
    
- Anonymous bind مفتوح
    

تقدر تسحب معلومات ضخمة عن الدومين.

---

### أداة ldapsearch

أداة بتعمل Query على LDAP.

مثال:

```
ldapsearch -x -H ldap://192.168.1.10 -b "dc=corp,dc=local"
```

-x = simple bind  
-H = السيرفر  
-b = base DN (جذر البحث)

الأداة بتعمل:

- تبعت LDAP query
    
- السيرفر يرجع Objects
    
- تطلع لك:
    
    - Users
        
    - Descriptions
        
    - SPNs
        
    - Group Membership
        

---

## الهجوم 2: LDAP Relay

ده امتداد لـ NTLM Relay  
لكن الهدف LDAP بدل SMB.

الأداة:

```
ntlmrelayx.py -t ldap://192.168.1.10
```

ليه ده خطير؟

لأن لو السيرفر قبل NTLM  
تقدر:

- تضيف User جديد
    
- تعدل ACL
    
- تضيف نفسك لـ Domain Admin
    

---

# 🔴 2) DCSync Attack (مهم جدًا)

دي من أخطر الهجمات في AD.

## الفكرة

Active Directory بيشتغل بنظام Replication  
يعني كل Domain Controller بينسخ بيانات من التاني.

فيه API اسمها:  
DRSUAPI

لو عندك صلاحية:  
Replicating Directory Changes

تقدر تقول للسيرفر:  
"اديني نسخة من Hashes كل المستخدمين"

---

## الأداة: Mimikatz

داخل Mimikatz:

```
lsadump::dcsync /domain:corp.local /user:Administrator
```

الأداة بتعمل:

- تستخدم DRSUAPI
    
- تطلب Hash المستخدم
    
- السيرفر يرده
    

ده مش بيحتاج تكون على DC  
بس لازم تكون:

- Domain Admin  
    أو
    
- عندك صلاحيات replication
    

---

# 🔴 3) Golden Ticket

دي هجمة Kerberos متقدمة.

## الأول نفهم حاجة:

في Kerberos فيه حساب اسمه:  
KRBTGT

ده الحساب اللي بيوقع TGT.

لو حصلت على:  
KRBTGT hash

تقدر تزور TGT بنفسك.

---

## الأداة: Mimikatz

```
kerberos::golden /domain:corp.local /sid:S-1-5-21-XXX /krbtgt:HASH /user:fakeadmin
```

الأداة بتعمل:

- تنشئ TGT مزيف
    
- توقّعه باستخدام KRBTGT hash
    
- تحقنه في الجلسة
    

بعدها:  
أي سيرفر يثق فيك.

---

# 🔴 4) Silver Ticket

زي Golden Ticket  
لكن بدل KRBTGT  
تزور Service Ticket لخدمة معينة (زي SMB).

أخف  
وأصعب في الاكتشاف.

---

# 🔴 5) SMB Exploits (زيادة مهمة)

مش بس مصادقة.

فيه ثغرات في SMB نفسه.

مثال:  
EternalBlue (MS17-010)

كان بيشتغل على:  
TCP 445

بيسمح Remote Code Execution.

الأداة:  
Metasploit module

---

# 🔴 6) ARP Packet Level (تفصيل أعمق)

ARP Packet فيه:

- Sender MAC
    
- Sender IP
    
- Target MAC
    
- Target IP
    

المهاجم يبعث:  
ARP Reply  
حتى لو محدش طلبه

السويتش يحدث ARP table  
ويبدأ الترافيك يروح للمهاجم.

---

# 🔴 7) VLAN Attack أعمق

802.1Q frame فيه:  
VLAN Tag

Double Tagging يحصل لما:

- أول tag يتشال
    
- التاني يكمل للـ VLAN الهدف
    

لكن ده بيشتغل بس لو:

- المهاجم في نفس VLAN الأصلية
    

---

# 🔴 8) SNMP Exploitation أعمق

SNMP بيستخدم:  
UDP 161

لو Community String = public

```
snmpwalk -v2c -c public 192.168.1.1
```

الأداة تبعت OID requests  
وترجع:

- Routing tables
    
- Interface data
    
- Device info
    

لو writeable community:  
تعدل config.

---

# 🔴 9) SMTP Abuse أعمق

بورت:  
25 / 587

الهجمات:

- Open Relay
    
- User Enumeration (VRFY command)
    
- Auth Brute Force
    

Hydra:

```
hydra -l user -P pass.txt smtp://192.168.1.5
```

---

# 🔴 10) SQL Injection أعمق

SQLmap بيعمل:

1. يحقن payload
    
2. يشوف error-based response
    
3. يستخرج DB schema
    
4. يعمل dump
    

هو بيبعت:  
HTTP Requests معدلة  
ويحلل الرد.

---

# 🔴 11) HTCP (زيادة — نادر في AD)

HTCP = Hyper Text Caching Protocol  
بيستخدم في Cache Proxies

بورت:  
UDP 4827

الهجوم:  
Cache poisoning

نادراً تشوفه في بيئات شركات عادية.

---

# 🔴 12) هجمات زيادة مهمة (مهم تعرفها)

## 1️⃣ BloodHound (أداة تحليل علاقات AD)

مش هجوم  
لكن بتحلل:  
ACLs  
Sessions  
Group membership

بتستخدم:  
Neo4j database

---

## 2️⃣ ACL Abuse

لو عندك:  
GenericAll  
WriteDACL

تقدر:

- تضيف نفسك لجروب
    
- تغير باسورد User
    
- تدي نفسك صلاحيات
    

---

## 3️⃣ Shadow Credentials (زيادة متقدمة)

استغلال msDS-KeyCredentialLink  
لعمل Certificate Authentication  
بدون باسورد.

---

# الصورة الكاملة الواقعية جدًا

في شغل حقيقي السيناريو بيبقى:

1. دخول VPN
    
2. LLMNR poisoning
    
3. NTLM Relay
    
4. User foothold
    
5. LDAP enum
    
6. BloodHound analysis
    
7. Kerberoast
    
8. Privilege escalation
    
9. DCSync
    
10. Golden Ticket
    
11. Persistence
    

---
