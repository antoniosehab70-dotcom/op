ممتاز 👌  
خلّينا نحل السؤال الأول **بفهم مش بحفظ**.

---

# ❓ Question:

**Which protocol is commonly targeted in password spraying attacks within networks?**

الاختيارات:

a) SMB  
b) ICMP  
c) SNMP  
d) SMTP

---

# ✅ الإجابة الصحيحة: **a) SMB**

لكن مش هنقولها ونمشي — هنفهم ليه 👇

---

# 🔥 الأول نفهم Password Spraying صح

## Password Spraying يعني إيه عمليًا؟

بدل ما أجرب:

```
user1 → 1000 password
```

أجرب:

```
500 users → password واحد
```

مثال:

```
Winter2024!
```

على كل يوزرات الشركة.

الهدف:

- أتفادى **Account Lockout**
    
- أستغل إن في موظفين كتير بيستخدموا باسورد متشابه
    

---

# 🧠 طيب أنا بجرب الباسورد ده فين؟

هنا السؤال المهم.

لازم أجربه على **خدمة بتقبل تسجيل دخول**.

في بيئة Windows Domain  
أشهر خدمة مصادقة بتقبل Login مباشر هي:

## 🔐 SMB (Server Message Block)

ده البروتوكول المسؤول عن:

- مشاركة الملفات
    
- مشاركة الطابعات
    
- Remote authentication
    
- بورت 445
    

وهو بيعتمد على:

- NTLM
    
- أو Kerberos
    

---

# ليه SMB هو الهدف الأشهر في Password Spraying؟

1. موجود في كل بيئة Windows تقريبًا
    
2. بيقبل Login مباشر
    
3. سهل تجرب عليه باستخدام أدوات زي:
    
    - CrackMapExec
        
    - Hydra
        
    - Impacket
        

مثال عملي:

```bash
crackmapexec smb 10.10.10.0/24 -u users.txt -p Winter2024!
```

ده Password Spraying على SMB.

---

# ❌ ليه مش باقي الاختيارات؟

## b) ICMP

ده بروتوكول Ping  
مفيهوش Authentication أصلاً  
مستحيل تعمل عليه Spraying.

---

## c) SNMP

بروتوكول إدارة أجهزة الشبكة  
أحيانًا فيه Community String  
لكن مش هو الهدف الشائع في بيئات AD.

---

## d) SMTP

بروتوكول إرسال إيميل  
ممكن تعمل عليه spraying  
لكن مش الأكثر شيوعًا في Windows Domain Networks.

---

# 💡 معلومة مهمة (إضافة مني — مهمة في الامتحانات)

في بيئات Active Directory  
أشهر 3 بروتوكولات بيتم عليها Password Spraying:

1. SMB (الأشهر)
    
2. OWA (Outlook Web Access)
    
3. LDAP
    

---


---



---



---

# 🔒 إضافة مهمة (مش ذكرتها قبل كده — مهمة)

لو الشركة مفعلّة:

- Account Lockout Policy
    
- Smart Lockout
    
- MFA
    

Password Spraying بيبقى أصعب.

عشان كده الشركات الحديثة:

- تراقب محاولات SMB Authentication
    
- تستخدم SIEM لاكتشاف محاولات spraying
    

---



---

# الخلاصة السريعة (تثبيت قبل الامتحان)

Password Spraying في بيئة Windows Domain  
أكثر بروتوكول يتم استهدافه =

👉 **SMB**

---

